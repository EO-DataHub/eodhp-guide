---
title: Credit rate derivation
doc_status: unreviewed
tags:
  - workspaces
last_reviewed:
reviewed_by:
review_notes: "Reconstructed from the deployed rates; the AWS list prices and the FX rate need confirming against the original working."
---
# Credit rate derivation

The rates in the deployed pricing policy recover what AWS charges the platform, and nothing more. Margin is applied separately, through the category multiplier. This page records how each rate is calculated, so the next recalibration is a repeat of a method rather than a fresh guess.

The rates it describes are those in `eodhp-argocd-deployment/apps/accounting-service/envs/*/products-prices-patch.yaml`. The `base/` values are a local development baseline and are not derived from anything.

## What a credit is

One credit is one pound. Nothing in `accounting-service` stores this. [D-revised](credits-ledger-design-decisions.md) removed the credit-to-currency rate from the schema, because no reader needed it and the service has no authority over a money value. The convention survives only here and in the `reason` field of each policy.

Every rate below is therefore a price in pounds, for one unit of the SKU.

## Inputs

The derivation takes four inputs. Record all four whenever you recalculate, because a rate cannot be checked without them.

| Input | Value used | Where it comes from |
|---|---|---|
| Region | `eu-west-2` (London) | Where the platform runs |
| Pricing model | On-demand list price | No reserved instances or savings plans are assumed |
| Exchange rate | 0.78 GBP per USD | AWS publishes eu-west-2 prices in USD |
| Reference instance | `m7i.xlarge` for CPU and memory, `g4dn.12xlarge` for GPU | The general-purpose and GPU node types in use |

AWS prices S3, EFS and data transfer per "GB", and defines that GB as 2^30 bytes. The collectors divide by `1024**3`, so the quantities and the prices agree. The `unit` strings in the configuration say `GB-s` and `GB`, which is the same misnomer AWS uses.

## Step 1: split the instance price into CPU and memory

The collector meters CPU and memory as two separate quantities, so one instance price has to become two rates. Splitting needs an assumed price ratio between a vCPU-hour and a GiB-hour.

Use 10:1. AWS Fargate, which is the only place AWS itself sells vCPU and memory separately, prices eu-west-2 at $0.04656 per vCPU-hour and $0.00511 per GB-hour, a ratio of 9.1:1. Rounding to 10 keeps the arithmetic legible and costs under 3%.

For `m7i.xlarge` at $0.2268 per hour, with 4 vCPU and 16 GiB:

```
0.2268 USD x 0.78            = 0.176904 GBP per hour
(4 x 10 + 16) = 56 memory-equivalents
0.176904 / 56                = 0.00315900 GBP per GiB-hour
0.00315900 x 10              = 0.03159000 GBP per vCPU-hour

memory-gb-seconds: 0.00315900 / 3600 = 8.775e-7
cpu-seconds:       0.03159000 / 3600 = 8.775e-6
```

The split is a convention, not a measurement. Any ratio recovers the same total for the reference instance. It changes only how the cost is shared between a workspace that asks for wide-and-shallow pods and one that asks for narrow-and-deep ones.

## Step 2: derive the GPU rate as a residual

The collector meters GPU count multiplied by time, and cannot tell which node type served the work. The rate has to be one flat number per GPU-second.

Take the GPU instance price, subtract its CPU and memory at the rates from step 1, and divide what is left by the number of GPUs. The subtraction matters: a GPU pod is already charged for its vCPUs and its memory, so charging the whole instance price again through `gpu-seconds` would bill those twice.

For `g4dn.12xlarge` at $4.632 per hour, with 48 vCPU, 192 GiB and 4 T4 GPUs:

```
4.632 USD x 0.78             = 3.61296 GBP per hour
48 x 0.03159                 = 1.51632 GBP  (CPU, already billed)
192 x 0.003159               = 0.60653 GBP  (memory, already billed)
3.61296 - 1.51632 - 0.60653  = 1.49011 GBP  (4 GPUs)
1.49011 / 4                  = 0.37253 GBP per GPU-hour

gpu-seconds: 0.37253 / 3600 = 1.0348e-4
```

## Step 3: convert storage prices from months to seconds

Both storage collectors sample a level, and the ingester integrates the samples into GB-seconds. AWS quotes a monthly price. Use AWS's own month of 730 hours, which is 2,628,000 seconds.

```
EFS Standard, 0.33 USD per GB-month:
  0.33 x 0.78 / 2628000  = 9.7945e-8

S3 Standard, 0.024 USD per GB-month:
  0.024 x 0.78 / 2628000 = 7.1233e-9
```

## Step 4: take requests and data transfer directly

These need only the exchange rate.

| SKU | AWS eu-west-2 list | Calculation | Result |
|---|---|---|---|
| `AWS-S3-API-CALLS` | $0.0004 per 1,000 GET | `0.0004 x 0.78 / 1000` | 3.12e-7 |
| `AWS-S3-DATA-TRANSFER-OUT-REGION` | free | — | 0 |
| `AWS-S3-DATA-TRANSFER-OUT-INTERREGION` | $0.02 per GB | `0.02 x 0.78` | 0.0156 |
| `AWS-S3-DATA-TRANSFER-OUT-INTERNET` | $0.09 per GB | `0.09 x 0.78` | 0.0702 |

Two of these carry an assumption worth stating.

`AWS-S3-API-CALLS` uses the GET price. AWS charges about 12 times more for a PUT. The collector reports one undifferentiated request count, so the rate has to pick a workload shape, and a data platform reads far more than it writes. A write-heavy workspace is undercharged, and the correct fix is to split the SKU rather than to raise the rate.

`AWS-S3-DATA-TRANSFER-OUT-REGION` is zero because same-region transfer out of S3 is genuinely free, not because the rate is unset. Leave the SKU in the policy: a missing rate and a zero rate behave differently, and a rated SKU keeps the usage visible on the Usage page.

## Step 5: round to one significant figure

Round each result to one significant figure. The error this adds is under 4%, against a 30% margin and an exchange rate that moves further than that in a year. False precision in a rate invites a reader to believe it is measured.

| SKU | Derived | Deployed | Error |
|---|---|---|---|
| `cpu-seconds` | 8.775e-6 | 9e-6 | +2.6% |
| `memory-gb-seconds` | 8.775e-7 | 9e-7 | +2.6% |
| `gpu-seconds` | 1.0348e-4 | 1e-4 | −3.4% |
| `EFS-STORAGE-STD` | 9.7945e-8 | 1e-7 | +2.1% |
| `AWS-S3-STORAGE` | 7.1233e-9 | 7e-9 | −1.7% |
| `AWS-S3-API-CALLS` | 3.12e-7 | 3e-7 | −3.8% |
| `AWS-S3-DATA-TRANSFER-OUT-INTERREGION` | 0.0156 | 0.015 | −3.8% |
| `AWS-S3-DATA-TRANSFER-OUT-INTERNET` | 0.0702 | 0.07 | −0.3% |

## Step 6: apply margin as a category multiplier

The rates stay at cost. Margin goes on the category multiplier. Keeping the two separate means an exchange-rate move and a commercial decision are different edits to different lines.

| Environment | `standard` | `academic` |
|---|---|---|
| test | 1 | 0.5 |
| staging | 1.3 | 1.3 |
| production | 1.3 | 1.3 |

Staging and production set both categories to 1.3, which was agreed with the funders because only one account type is in use. Two consequences follow.

The category is inert where it is deployed. Every workspace prices the same whatever category it holds, so a wrong category assignment cannot be detected from a bill. When a real academic tier arrives, it needs a new policy version rather than an edit to a live one.

Test is not a rehearsal of production pricing. It carries the cost-recovery rates with no margin and a real academic discount, so a credit total observed on test will not match the same usage on production. Compare the two by dividing out the multiplier, not by comparing totals.

## What the rates do not cover

The rates recover the AWS charges that the collectors can attribute to a workspace. Several real costs fall outside that.

- **Unused node capacity.** The collector bills `max(used, requested)` per pod, so a pod that reserves and idles is charged. Nothing charges for a node's spare capacity, its system DaemonSets, or a node kept warm for scheduling. At 60% cluster utilisation, a 1.3 multiplier recovers cost and no margin.
- **Network address translation.** A NAT gateway costs about $0.045 per GB processed, plus an hourly charge. Traffic from a workspace pod to the internet crosses one and is not metered. A gateway VPC endpoint makes S3 traffic bypass it; confirm one exists before treating in-region S3 reads as free end to end.
- **Shared platform infrastructure.** The EKS control plane, load balancers, the Pulsar cluster, the databases and the harvesters are platform costs with no per-workspace quantity to attribute them to.
- **EFS throughput.** The rate covers Standard storage. Elastic throughput mode adds a per-GB charge for reads and writes, and provisioned throughput adds a flat monthly charge. Check which mode the file systems use.
- **Storage classes other than Standard.** S3 Infrequent Access, Glacier and EFS-IA all price differently, and one SKU each cannot express that.

## What the rates imply in practice

A recalibration should be checked against usage, not only against a spreadsheet. These are the figures the current rates produce, before the multiplier.

| Scenario | Credits |
|---|---|
| 2 vCPU / 8 GiB notebook, 20 hours a month, 20 GiB EFS, 5 GiB S3 | 7.3 per month |
| 8 vCPU / 32 GiB notebook, 120 hours a month, 200 GiB EFS, 500 GiB S3 | 109 per month |
| One T4 GPU with 4 vCPU / 16 GiB, 8 hours | 4.3 |
| Idle workspace holding 100 GiB EFS and 1 TiB S3 | 45 per month |

Storage dominates. EFS at £0.26 per GiB-month is 14 times the price of S3, and an idle workspace holding 100 GiB of it costs more per month than a heavy user's compute. That is AWS's pricing, correctly reflected, but it makes quota policy on EFS more consequential than any compute rate.

## When to recalculate

Recalculate when the exchange rate moves more than about 10%, when AWS changes its list prices, or when the platform adopts a node type that the reference instances no longer represent. Record the four inputs from the table above in the policy's `reason` field. A rate whose working cannot be found is a rate nobody can defend.
