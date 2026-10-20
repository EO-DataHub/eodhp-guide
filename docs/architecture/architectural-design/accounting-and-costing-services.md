---
title: 3.14 Accounting and Costing Services
doc_status: ok
last_reviewed:
reviewed_by:
review_notes:
tags:
  - workspaces
  - aws
---
### 3.14 Accounting and Costing Services


```kroki-plantuml
@startuml


package "Web Presence" {
  [Workspace UI] as WorkspaceUI
}

package "Workspace Storage" as UserStorage {
  [S3] as S3Storage
  [NFS] as NFSStorage
}
  
package "Workspace Management" as WSMGMT {
}

package "Workflow and Analysis System" as WAS {
  node "K8S Workspace\nNamespace" as WorkspaceNamespace {
  }
}

node CloudFront {
  [Workspaces CF\nDistribution] as WorkspaceCF
}

node S3 {
  database "CloudFront\nAccess Logs" as CFLogs
  database "S3\nAccess Logs" as S3Logs
}

S3Storage --> S3Logs
WorkspaceCF --> CFLogs

package "Accounting" {
  [Accounting\nAPI] as API
  [Accounting\nIngester] as Ingester
  [billing-admin\nCLI] as AdminCLI
  [EFS\nCollector] as EFSCollector
  [S3\nCollector] as S3Collector
  [Data Transfer\nCollector] as EgressCollector
  [Compute\nCollector] as ComputeCollector

  database "AccountingData\n(usage, pricing policies,\ncredit ledger)" as AccountingData

  API <-- AccountingData
  Ingester --> AccountingData : Usage and\ndebits
  AdminCLI --> AccountingData : Grants, categories,\nrecharges


  database "Products and\nPricing Policy\n(ConfigMap)" as PriceData
  Ingester <-- PriceData
}

actor "Hub Admin" as HubAdmin
HubAdmin --> AdminCLI

interface AccountingAPI
WorkspaceUI <-- AccountingAPI : Rates, usage, credit\nbalance and ledger
AccountingAPI -- API

Ingester <... WSMGMT : Workspace lifecycle data\n(via msg)
ComputeCollector <-- WorkspaceNamespace
ComputeCollector ..> Ingester : CPU/Memory data\n(via msg)

CFLogs --> EgressCollector
EgressCollector ..> Ingester : HTTPS-based bandwidth use data\n(via msg)

S3Logs --> S3Collector : Access data
S3Storage --> S3Collector : Size samples
S3Collector ..> Ingester : S3 storage and API use data\n(via msg)

NFSStorage --> EFSCollector : Size samples
EFSCollector ..> Ingester : EFS storage use data\n(via msg)

@enduml
```
**Figure 3-19 Accounting Service (arrows show data flows)** 

The Accounting Service consists of

- Collectors, which collect specific billing data and send it to Pulsar.
- The Ingester, which receives billing data from Pulsar, records it, and charges it in credits.
- The API, which serves usage, pricing and credit data over a read-only API.
- The `billing-admin` CLI, which hub admins use to grant credits, set workspace pricing categories and repair data. It connects directly to the database.
- A PostgreSQL database called ‘accounting’, which holds usage data, versioned pricing policies and an append-only credit ledger.
- A Kubernetes ConfigMap, configured using Git and ArgoCD, which sets the products and the pricing policy.

Usage is charged in **credits**. Credits are the platform's unit of account. Nothing converts credits to money: a hub admin grants credits to a workspace, and usage spends them.

#### 3.14.1 Ingester

On startup, the Ingester loads the ConfigMap. This holds two things:

- The products available (name, SKU and priced unit such as GB-s). Accounting collectors specify the SKU for the data they are collecting.
- The pricing policy: a credit rate for each SKU, a multiplier for each workspace pricing category, and the default category. See [3.14.3 Pricing Policies](#3143-pricing-policies).

The Ingester receives messages of several types and updates the accounting database

- Billing Events, which describe consumption of a certain amount of a resource (product) over a particular time window. All billing events are specific to a workspace and product. These never cross day boundaries.
- Resource Consumption Rate Samples, which are samples of the rate at which a resource is being consumed from which the Ingester will estimate Billing Events. These are typically storage consumption rates – a snapshot of the size of a data store in GB is a sample of the rate at which a storage product (priced in GB-s) is being consumed.
- Workspace settings messages, which allow the Accounting API to know which Workspaces are in which Billing Accounts.

For every Billing Event it records, whether received directly or estimated from Resource Consumption Rate Samples, the Ingester writes one debit to the credit ledger. The event and its debit are written in one database transaction. A unique index allows only one original debit per event, so a redelivered message is not charged twice.

The Ingester records an event without a debit, and logs an error, when:

- no pricing policy covers the time of the usage,
- the policy has no rate for the SKU, or
- the quantity is negative or not finite.

Redelivering the message does not fix any of these, so the message is not failed. The quantity is kept, and `billing-admin recharge` can charge it later.

#### 3.14.2 Collectors

The Collectors each function differently and the system is extensible to new Collectors.

The Compute Collector uses data collected by Prometheus about CPU and memory consumption in workspace namespaces to generate (exact) Billing Events.

The EFS Collector mounts the EFS volume holding workspace block stores and takes periodic samples of storage use.

The S3 Collector generates two streams of data about workspace object stores. It uses AWS Athena to process S3 logs and generate Billing Events for the number of S3 API calls made and the egress bandwidth used by them (which is split according to destination – regional, inter-regional or internet). It also takes storage use samples and submits them as Resource Consumption Rate Samples.

The Data Transfer Collector parses CloudFront access logs and generates Billing Events for data egress. This covers HTTPS-based access to workspace stores and is split according to destination.

Collectors are designed to recover after downtime by recording their last processed position and/or generating data starting some period before their startup time. In the second case, Billing Events are generated with UUIDs which will be identical for identical (time period, product, workspace) tuples. The Ingester discards duplicate messages. This is not possible with Resource Consumption Rate Sample messages because the data is not available retrospectively, but Billing Events will still be generated through interpolation just with degraded accuracy.

#### 3.14.3 Pricing Policies

A pricing policy holds every credit rate and every category multiplier together, as one version. Rates are never changed one at a time.

When the Ingester loads the ConfigMap, it compares the `pricing_policy` section with the policy in force. If a rate, a multiplier, the default category or `valid_from` has changed, it creates a new policy version. Otherwise it writes nothing. A changed `reason` alone does not create a version.

The charge for one Billing Event is:

```
credits = quantity × credits_per_unit (for the SKU) × multiplier (for the workspace's category)
```

The policy used is the one in force at the event's start time. The workspace category is the one assigned at the time of charging. A workspace with no category is charged under the policy's default category. Charges are stored exactly, with no rounding.

Policies are resolved by the time they were configured, not only by `valid_from`. A backdated policy therefore applies to every date from its own `valid_from` onwards, and replaces any policy configured before it for those dates.

The workspace service will become the authority for workspace categories and will send them over Pulsar. Until then, hub admins set categories with `billing-admin set-category`.

#### 3.14.4 Credit Ledger

The credit ledger records every movement of credits for a workspace:

- **Debits** (negative) for usage, written by the Ingester.
- **Grants** (positive), written by hub admins with `billing-admin grant`. Each grant records who made it and why.
- **Reversals**, which correct an earlier transaction.

The ledger is append-only. Nothing updates or deletes a row: a correction is a new row that references the row it corrects. A workspace's balance is the sum of its ledger. A negative balance is reported, but nothing in the platform blocks work because of it.

Each debit stores the quantity metered, the pricing policy version that priced it, and the workspace category in force at the time. A charge can therefore be explained months later, after the rates have changed or the workspace has moved to another category.

A balance snapshot table exists to make balance reads cheaper in future, for example for budget checks. Nothing writes to it yet.

#### 3.14.5 API

The Accounting API serves products, pricing, usage and credit data via a read-only API. The Workspace UI uses it to show per-Workspace resource consumption, the credits it cost, and the workspace's balance and transactions.

Every endpoint requires a Keycloak token. The API verifies the token signature itself against Keycloak's published keys, and accepts the `eodh` and `eodh-workspaces` audiences. It does not rely on the gateway to have checked the token.

- Workspace endpoints (usage, balance, ledger) need membership of the workspace.
- Account usage endpoints need ownership of the billing account.
- Product and pricing endpoints need any valid token.
- Hub admins pass every check, and only hub admins can read the pricing policy in force at a past instant.

Usage reads report both the quantity consumed and the credits it cost. They can be totalled by day, week, month or quarter (calendar periods in UTC), filtered by user or SKU, and grouped by user, SKU or workspace. Usage reads net a reversal against the charge it corrects. The ledger endpoints list every movement as recorded, including reversals.

See [Accounting](../../reference/services/accounting.md) for the endpoint list and admin procedures.

#### 3.14.6 Kubecost and Workspace Cost Visibility

Kubecost is installed in the cluster and provides Kubernetes-native cost allocation data — CPU, memory, persistent volume, network, and load balancer costs — aggregated per namespace. This is complementary to the Accounting Service:

| | Accounting Service | Kubecost |
|-|--------------------|---------|
| **Data source** | Collectors (S3 logs, EFS samples, CloudFront logs, Prometheus) | Kubernetes resource metrics |
| **Purpose** | Usage metering and credit charging | Operational cost visibility |
| **Granularity** | Per product/SKU per workspace | Per namespace, broken down by resource type |
| **Status** | Implemented: usage charged in credits against a ledger | Installed in cluster |

The **Workspace Cost API** (`kubecost-eodh-api`) sits in front of Kubecost and enforces workspace membership scoping so that users only see cost data for namespaces belonging to their own workspaces. It exposes two endpoints: a per-workspace total cost summary and a detailed per-namespace breakdown with top cost drivers.

This API is in development and not yet live. See [Workspace Cost API](../../explanation/architecture/workspace-cost-api.md) for the full design and API reference.
