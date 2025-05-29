# Accounting

## Summary

This

- Collects/generates and stores resource use data relevant to billing.
- Knows product and price settings.
- Serves accounting, product and price data to users.

The accounting subsystem consists of several related microservices linked by messaging - a central Accounting Service and multiple Collectors.

These messages are defined in eodhp-utils, in eodhp_utils/pulsar/messages.py.

## Data Model and Database

The data model consists of:

- Billing Events, which record resource consumption by a particular workspace, over a particular time period (typically 5 minutes, 1 hour or 1 day), and of a particular product/resource.
- Resource Consumption Rate Samples, which are point-in-time samples of the rate at which a particular product is being consumed by a particular workspace.
- Products, consisting of an SKU (stock-keeping unit, a human-readable identifier), a name and the units in which consumption is measured.
- Prices, which give the price of a unit of a particular product which applied between particular dates.

These are all stored persistently in AWS Aurora in a database called 'accounting'. Prices and products can be modified there if necessary but should usually be modified via the configuration file.

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

#### Operation

The API Service is a Kubernetes Service and Deployment, both called `accounting-api` in the `accounting` namespace. The Ingester is a Deployment only, called `accounting-ingester`. Both may and do have multiple replicas.

The accounting API service serves `/api/accounting/prices` and `/api/accounting/skus` with no authentication requirement. For the other endpoints (see `/api/docs`) either the `hub_admin` role is required, or for `/api/workspaces/<workspace>/accounting/...` ownership or membership of the workspace is required, or for `/api/workspaces/<account>/accounting` ownership of the account is required.

#### Configuring Products and Prices

Products and prices can be configured via a ConfigMap, which is created at
`apps/accounting-service/base/products-prices-config.yaml` in eodhp-argocd-deployment. For example

```
    items:
      - sku: cpu-seconds
        name: CPU use by workspace pods
        unit: "s"
    prices:
      - sku: cpu-seconds
        valid_from: "2025-01-02T00:00:00Z"
        price: 0.0000012
```

For billing items `sku` is a human-readable and potentially user-visible identifier which must match that used by the billing collector generating billing events for the product. This is a fixed and hard-coded value and all known values are listed in the initial ConfigMap. `name` and `unit` are for display to users and do not affect functionality.

When setting prices `price` should be the price in pounds charged for one `unit` of the resource. `valid_from` sets the time from which this price will be applied to new consumption, with prior consumption still being billed at an earlier price (even if the bill has not been issued yet).

`valid_from` should usually be changed to a future time when changing a price. Not doing so will cause billing history to be rewritten, changing the billed amount for past consumption of the product. This is useful only for correcting mistakes.

Items will be auto-created if a billing event is observed with an unknown SKU. These will have an empty name and unit (and no prices) and these can be updated later via the config file. This is to avoid data loss in the event of misconfiguration.

After changing this file it must be deployed to the cluster and the accounting ingester must be restarted.

Neither prices nor items can be deleted using this file.

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

The sampling frequency can be set with the `--interval` command line option in the Deployment in ArgoCD repo (apps/accounting-service/base/efs-sampler-deployment.yaml).

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

The Job can be triggered using the 'Create Job' in the ArgoCD UI, shown for the cronjob in the Accounting app. Jobs can deleted there or using `kubectl` to abandon them.

This Collector records its progress into the volume attached to the `accounting-logs-cache-claim-efs` PVC. It will restart at the last log file it successfully processed. A locking mechanism is used to prevent race conditions.

#### Dependencies

Depends on AWS's S3 and Athena, and on Pulsar. Also depends on an IAM role called `AccountingS3Collector-<env name>` set up by ArgoCD.
