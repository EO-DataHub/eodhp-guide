---
title: Accounting
doc_status: ok
last_reviewed:
reviewed_by:
review_notes: Migrated from docs/services/accounting.md (no split).
tags:
  - workspaces
  - aws
---
# Accounting

## Summary

The accounting subsystem:

- Collects/generates and stores resource use data relevant to billing.
- Knows product settings and the pricing policy (credit rates and category multipliers).
- Charges usage in credits and keeps an append-only credit ledger for each workspace.
- Serves usage, product, pricing and credit data to users.

The accounting subsystem consists of several related microservices linked by messaging - a central Accounting Service and multiple Collectors.

These messages are defined in eodhp-utils, in eodhp_utils/pulsar/messages.py.

## Data Model and Database

The data model consists of:

- Billing Events, which record resource consumption by a particular workspace, over a particular time period (typically 5 minutes, 1 hour or 1 day), and of a particular product/resource.
- Resource Consumption Rate Samples, which are point-in-time samples of the rate at which a particular product is being consumed by a particular workspace.
- Products, consisting of an SKU (stock-keeping unit, a human-readable identifier), a name and the units in which consumption is measured.
- Pricing Policies, versioned. Each version holds a credit rate for every SKU, a multiplier for every workspace pricing category, and a default category.
- Workspace Categories, which select the multiplier a workspace is charged under.
- Credit Ledger Transactions: usage debits (negative), admin grants (positive) and reversals. The ledger is append-only. A workspace's balance is the sum of its ledger.
- Credit Balance Snapshots, an optimisation for balance reads. Nothing writes to this table yet.

These are all stored persistently in AWS Aurora in a database called 'accounting'. The schema is managed with Alembic. A Kubernetes Job applies migrations before a new version starts.

Products and pricing policies should be changed through the configuration file. Grants and categories are changed with the `billing-admin` CLI (see [Managing Credits](#managing-credits)).

Credits are the unit of account. Nothing converts credits to money.

The Ingester charges every Billing Event it records, including those estimated from Resource Consumption Rate Samples. Each debit stores the quantity, the policy version and the workspace category used, so it can be explained later.

The Ingester will generate Billing Events from Resource Consumption Rate Samples via linear interpolation to the boundaries of one hour intervals. This is done only for S3 and EFS storage use, for all other products exact Billing Events are generated directly by Collectors.

Billing Events have UUIDs and messages with duplicate UUIDs will be ignored. Collectors make use of this by generating UUIDs based on the time period, workspace and product they're generating events for.

## Services, Repositories and Artefacts

The deployment of all services except for the Compute Collector is configured in the https://github.com/EO-DataHub/eodhp-argocd-deployment repository, apps/accounting directory. The product and price data is also configured there in apps/accounting-service/base/products-prices-config.yaml .

The Compute Collector deployment is configured in the same repository but at apps/billing-collector.

### Microservice: accounting service

#### Code Repositories and Artifacts

- Code at https://github.com/EO-DataHub/accounting-service
- Container image pushed to public.ecr.aws/eodh/accounting-service

This microservice consists of two services sharing a database and codebase - the API service and the ingester.

#### Dependent Services

The Workspace UI is dependent on the API Service. There are multiple replicas of this service.

The API Service depends on the Ingester adding data to the database but will still serve existing data without it.

The Ingester depends on Collectors to send it data and the Collectors on the Ingester to store it. These are linked by Pulsar (asynchronous persistent messaging) and none will fail due to temporary downtime of any other.

The ingester uses a Pulsar schema for the workspace-settings topic whereas the workspace-manager which writes to this topic does
not. If a schema compatibility error is observed then the following may solve it:

```
kubectl port-forward service/pulsar-proxy -n pulsar 8080:8080 &
pulsar-admin --admin-url http://localhost:8080 topicPolicies set-schema-compatibility-strategy -s=ALWAYS_COMPATIBLE persistent://public/default/workspace-settings
```

`pulsar-admin` comes with the Pulsar binary download.

This is only likely to be seen in new clusters and may only happen if service startup and first workspace status
message have a particular order.

#### Operation

The API Service is a Kubernetes Service and Deployment, both called `accounting-api` in the `accounting` namespace. The Ingester is a Deployment only, called `accounting-ingester`. Both may and do have multiple replicas.

Every endpoint needs a Keycloak bearer token. The API verifies the token signature against Keycloak's published keys (`KEYCLOAK_BASE_URL` must be set) and accepts the `eodh` and `eodh-workspaces` audiences. Users with the `hub_admin` realm role pass every check.

| Endpoint | Access | Returns |
|----------|--------|---------|
| `/api/accounting/skus`, `/api/accounting/skus/{sku}` | Any valid token | Billing items (products) |
| `/api/accounting/prices` | Any valid token | Credits per unit for every SKU, with the `valid_from` and `policy_version` they came from. Hub admins may pass `at` for a past instant |
| `/api/accounting/pricing-policy` | Any valid token | The whole policy in force: rates, category multipliers and default category. Hub admins may pass `at` |
| `/api/workspaces/{workspace}/accounting/usage-data` | Workspace member | Usage, with `quantity` and `credits` per row |
| `/api/workspaces/{workspace}/accounting/balance` | Workspace member | The credit balance, as an exact decimal string |
| `/api/workspaces/{workspace}/accounting/ledger` | Workspace member | Credit transactions, newest first. Filter with `type=grant`, `debit` or `reversal`, and `start`/`end`. Page with `limit` and `after` |
| `/api/workspaces/{workspace}/accounting/ledger/{transaction}` | Workspace member | One transaction. For a debit, `pricing` gives the quantity, credits per unit, category, multiplier and policy version |
| `/api/accounts/{account_id}/accounting/usage-data` | Billing account owner | Usage for every workspace in the account |

The usage endpoints take these query parameters:

- `start`, `end`: time range.
- `limit`, `after`: paging. `limit` must be 1 or more.
- `time-aggregation`: `day`, `week`, `month` or `quarter`. Periods are calendar periods in UTC. A week starts on Monday. Any other value, including an empty one, returns 422.
- `user`: only usage attributed to this user UUID. Workspace storage has no user, so this filter leaves it out.
- `sku`: only usage of this SKU.
- `group-by`: any of `user`, `sku` and `workspace`, comma-separated. The default is `sku,workspace`. Needs `time-aggregation`.

Usage reads net a reversal against the charge it corrects. The ledger endpoints list every movement as recorded.

Decimal values (rates, credits, balances) are exact decimal strings, for example `"0.000000412"`. They are not rounded. Clients format them for display.

#### Configuring Products and Pricing Policy

Products and the pricing policy are configured in the `products-prices` ConfigMap, at
`apps/accounting-service/base/products-prices-config.yaml` in eodhp-argocd-deployment. The ConfigMap holds an `accounting.conf` document. For example

```yaml
items:
  - sku: cpu-seconds
    name: Notebook and workflow CPU time
    unit: "s"

pricing_policy:
  valid_from: "2025-01-01T00:00:00Z"
  default_category: standard
  reason: "Why this calibration happened"
  rates:
    - sku: cpu-seconds
      credits_per_unit: 0.001
  category_multipliers:
    - category: standard
      multiplier: 1
    - category: academic
      multiplier: 0.5
```

For billing items `sku` is a human-readable and potentially user-visible identifier which must match that used by the billing collector generating billing events for the product. This is a fixed and hard-coded value and all known values are listed in the initial ConfigMap. `name` and `unit` are for display to users and do not affect functionality.

The `pricing_policy` section sets every rate and every multiplier together:

- `credits_per_unit` is the number of credits charged for one `unit` of the SKU.
- `category_multipliers` scale every rate for workspaces in that category.
- `default_category` is used for workspaces with no category. It must have a multiplier.
- `valid_from` is the time from which the policy applies.
- `reason` is recorded with the policy for audit.

When the Ingester starts, it compares this section with the policy in force. It creates a new policy version only when a rate, a multiplier, the default category or `valid_from` changes. Restarting with the file unchanged writes nothing. Changing only `reason` writes nothing.

Charges already written keep the rate and policy version that priced them. A policy is resolved by when it was configured: a backdated policy applies to every date from its `valid_from` onwards, and replaces older policies for those dates. Set `valid_from` to a future time for a normal rate change. Backdate only to correct a mistake.

Items will be auto-created if a billing event is observed with an unknown SKU. These will have an empty name and unit (and no rate) and these can be updated later via the config file. This is to avoid data loss in the event of misconfiguration. An event for a SKU with no rate is recorded but not charged, and an error is logged.

After changing this file it must be deployed to the cluster and the accounting ingester must be restarted.

Neither policies nor items can be deleted using this file.

#### Managing Credits

The `billing-admin` CLI is in the accounting-service repository (`uv run billing-admin --help`). It connects directly to the database, so first point it at the right one, for example through a `kubectl port-forward` and the `SQL_*` environment variables. Write commands do not ask for confirmation.

| Command | Purpose |
|---------|---------|
| `billing-admin ls [SKU]` | List every item with its rate under the policy in force, or one SKU's rate in every policy version |
| `billing-admin add-item`, `update-item` | Create an item, or fix an item's name and unit without a redeploy |
| `billing-admin grant -w WORKSPACE -a AMOUNT -r REASON` | Grant credits to a workspace. Not idempotent: running it twice grants twice |
| `billing-admin set-category -w WORKSPACE -c CATEGORY` | Set a workspace's pricing category. Affects future charges only |
| `billing-admin ledger WORKSPACE` | Show recent transactions and the balance |
| `billing-admin workspaces [--unmapped]` | List workspaces that have usage or an account mapping, one per line. `--unmapped` lists only workspaces with usage but no account mapping |
| `billing-admin recharge [--commit]` | Charge events that were recorded but never charged, for example usage from before any policy existed. Dry run unless `--commit` is given. Safe to re-run. Filter with `--start`, `--end`, `--workspace`, `--sku`, and pin a version with `--policy` |

Users who need credits ask a hub admin, who grants them. Until the workspace service sends categories over Pulsar, `set-category` is the only way to set a category, and every other workspace is charged under the default category.

#### Control

Both services in this microservice can be restarted without downtime using `make krestart` at the root of the `accounting-service` Git repo. Alternatively, use

- `kubectl rollout restart deployment.apps/accounting-api -n accounting`
- `kubectl rollout restart deployment.apps/accounting-ingester -n accounting`

The same can be accomplished in the Accounting app in the ArgoCD UI.

Deleting the Pods or Deployments will also work, but with downtime.

#### Dependencies

The Accounting API and Accounting Ingester depend on the AWS Aurora DB and will not start without it. The Accounting Ingester also depends on Pulsar and will not start without it.

### Microservice: Compute Collector

This gathers CPU and memory use data about workspace namespaces.

#### Code Repositories and Artifacts

- Code at https://github.com/EO-DataHub/billing-collector
- Container image pushed to public.ecr.aws/eodh/billing-collector

#### Dependent Services

None

#### Operation

This service runs as a Kubernetes Deployment called `billing-collector` in the `billing-collector` namespace.

#### Configuration

The frequency with which Billing Events are generated can be set as an environment variable in the Deployment in ArgoCD repo (apps/billing-collector/base/deployment.yaml).

#### Control

To restart use `kubectl rollout restart -n billing-collector deployment billing-collector` or use the 'Restart' option for the billing-collector Deployment in the Billing Collector app in the ArgoCD UI.

The collector will restart collection from 1 hour before its start time, relying on UUID-based deduplication at the Ingester. Up to 1 hour of downtime is possible without data loss.

To generate Billing Events for earlier time windows, the collector can be run with the `--from <ISO8601 timestamp>` command-line option. It will exit when it has caught up, so this must either be run as a separate Kubernetes Job or the option must be removed once it has run.

#### Dependencies

This service depends on Prometheus and (through it) the Kubernetes cadvisor APIs.

### Microservice: EFS Collector

#### Code Repositories and Artifacts

- Code at https://github.com/EO-DataHub/eodhp-accounting-efs
- Container image pushed to public.ecr.aws/eodh/accounting-efs

#### Dependent Services

None

#### Operation

This service runs as a Kubernetes Deployment called `accounting-efs-sampler-collector` in the `accounting` namespace. The EFS volume used for workspace block stores is mounted into it (read-only) and the collector periodically calculates disk space use for each workspace.

#### Configuration

The sampling frequency can be set with the `--interval` command line option in the Deployment in ArgoCD repo (apps/accounting-service/base/efs-sampler-deployment.yaml).

#### Control

To restart use `make krestart` in the root of the Git repo, or `kubectl rollout restart deployment.apps/accounting-efs-sampler-collector -n accounting`, or use the 'Restart' option for the Deployment in the Accounting app in the ArgoCD UI.

Extended downtime may result in degraded accuracy when billing for EFS storage use.

#### Dependencies

Depends on AWS's EFS and on Pulsar and will not start without both.

### Microservice: S3 Collector

#### Code Repositories and Artifacts

- Code at https://github.com/EO-DataHub/eodhp-accounting-s3-usage
- Container image pushed to public.ecr.aws/eodh/accounting-s3-usage

#### Dependent Services

None

#### Operation

This service runs as a Kubernetes Deployment called `accounting-s3-collector` in the `accounting` namespace.

This:

- Samples S3 storage use, and
- Uses the `accounting_eodhp_<env name>` AWS Athena DB and `accounting-athena-<env name>` S3 bucket to process the S3 logs written to `s3://workspaces-access-logs-<env-name>/s3/standard/`.

#### Configuration

The sampling frequency can be set with the `--interval` command line option on the Deployment for `accounting-s3-collector` under `apps/accounting-service/base/` in `eodhp-argocd-deployment` (see that repository for the current manifest filename—the older doc copy mistakenly referenced `efs-sampler-deployment.yaml` here).

The backfill time can be set with the `--backfill` option. This causes the Collector to start collecting this many intervals prior to its start time, relying on UUID-based deduplication.

#### Control

To restart use `make krestart` in the root of the Git repo, or `kubectl rollout restart deployment.apps/accounting-s3-collector -n accounting`, or use the 'Restart' option for the Deployment in the Accounting app in the ArgoCD UI.

Extended downtime may result in degraded accuracy when billing for S3 storage use. S3 bandwidth use and API call use can be recovered, providing `--backfill` is large enough to cover the gap.

#### Dependencies

Depends on AWS's S3 and Athena, and on Pulsar. Also depends on an IAM role called `AccountingS3Collector-<env name>` set up by ArgoCD.

### Microservice: Data Transfer Collector

#### Code Repositories and Artifacts

- Code at https://github.com/EO-DataHub/eodhp-data-transfer-events
- Container image pushed to public.ecr.aws/eodh/eodhp-accounting-cloudfront

#### Dependent Services

None

#### Operation

This service runs as a Kubernetes CronJob called `accounting-logs` in the `accounting` namespace.

This reads and processes CloudFront logs written to the `access-logs-<env name>` S3 bucket. It generates bandwidth consumption events for HTTPS downloads from workspace domain names.

#### Configuration

The run frequency can be set in the CronJob in the ArgoCD repo (apps/accounting-service/base/accounting-logs.yaml).

#### Control

The Job can be triggered using the 'Create Job' in the ArgoCD UI, shown for the cronjob in the Accounting app. Jobs can be deleted there or using `kubectl` to abandon them.

This Collector records its progress into the volume attached to the `accounting-logs-cache-claim-efs` PVC. It will restart at the last log file it successfully processed. A locking mechanism is used to prevent race conditions.

#### Dependencies

Depends on AWS's S3 and Athena, and on Pulsar. Also depends on an IAM role called `AccountingS3Collector-<env name>` set up by ArgoCD.
