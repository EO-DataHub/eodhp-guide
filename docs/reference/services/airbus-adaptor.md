---
title: Airbus Adaptor
doc_status: unreviewed
tags:
  - commercial-data
  - stac
  - workflows
last_reviewed:
reviewed_by:
review_notes:
---
# Airbus Adaptor

Reference for the Airbus commercial data adaptors, which order satellite imagery from the Airbus OneAtlas API and deliver it into an EO DataHub workspace. Two separate adaptors are provided — one for optical imagery and one for SAR (Synthetic Aperture Radar) — each running as a short-lived Kubernetes pod orchestrated by a CWL workflow.

**How-to:** [Test commercial data adaptor changes](../../how-to/data-and-catalogues/test-commercial-data-adaptor.md) · **Explanation:** [Commercial Data Purchasing Pipeline](../../explanation/architecture/commercial-data-purchasing.md)

For operator-level secrets setup (OTP XOR mechanism, kubectl sanity checks), see [Data Adaptors — Airbus workspace secrets](data-adaptors.md#airbus-workspace-secrets-operator-reference).

## Execution Model

Two CWL workflows are provided:

- **`airbus-optical.cwl`** — wraps a `CommandLineTool` that pulls `public.ecr.aws/eodh/airbus-optical-adaptor:0.0.7` and invokes `python -m airbus_optical_adaptor`.
- **`airbus-sar.cwl`** — wraps a `CommandLineTool` that pulls `public.ecr.aws/eodh/airbus-sar-adaptor:0.0.11` and invokes `python -m airbus_sar_adaptor`.

Both inject `CLUSTER_PREFIX` as an environment variable and produce the working directory (`.`) as their output, which contains the STAC records written during execution.

A third workflow, **`airbus-optical-multi.cwl`**, accepts an array of STAC directories (`stac_keys`) for batch optical orders but is not in active use.

- **Source code:** [EO-DataHub/commercial-data-adaptors](https://github.com/EO-DataHub/commercial-data-adaptors)

## Inputs

### Optical (`airbus-optical.cwl`)

| Parameter | Description |
|---|---|
| `workspace` | Workspace identifier within the platform |
| `workspace_bucket` | S3 bucket where workspace STAC records are stored |
| `commercial_data_bucket` | S3 bucket where Airbus delivers order archives |
| `pulsar_url` | Apache Pulsar broker URL for downstream event messaging |
| `cluster_prefix` | Platform environment prefix (injected as `CLUSTER_PREFIX`) |
| `product_bundle` | Named bundle controlling processing parameters: `Visual`, `General Use`, `Analytic`, or `Basic` |
| `coordinates` | Area of interest coordinates (JSON-stringified) |
| `stac_key` | Directory containing one or more STAC catalogs describing items to order |
| `end_users` | JSON-stringified list of end-user names and nationalities — required for PNEO orders |
| `licence` | Licence identifier applied to the order |

### SAR (`airbus-sar.cwl`)

| Parameter | Description |
|---|---|
| `workspace` | Workspace identifier within the platform |
| `workspace_bucket` | S3 bucket where workspace STAC records are stored |
| `commercial_data_bucket` | S3 bucket where Airbus delivers order archives |
| `pulsar_url` | Apache Pulsar broker URL for downstream event messaging |
| `cluster_prefix` | Platform environment prefix (injected as `CLUSTER_PREFIX`) |
| `product_bundle` | JSON-stringified object containing `product_type` (SSC/MGD/GEC/EEC), `orbit` (rapid/science), `resolution` (RE/SE), and `projection` (auto/UTM/UPS) |
| `coordinates` | Area of interest coordinates (JSON-stringified) |
| `stac_key` | Directory containing one or more STAC catalogs describing items to order |
| `licence` | Order template identifier applied to the order |

## Processing Pipeline

Both adaptors share the same high-level pipeline, with sensor-specific differences noted below.

### Optical

For each STAC item found in the input catalogs, the adaptor executes the following steps in sequence, aborting the item and writing a failure record on any error:

1. **Authenticate** — Retrieves an OTP key from a Kubernetes Secret (`otp-airbus` in the workspace namespace), fetches the corresponding encrypted API key from AWS Secrets Manager, and decrypts it using XOR One-Time Pad. The plaintext key is exchanged for a short-lived OAuth2 bearer token via the OneAtlas IDP at `authenticate.foundation.api.oneatlas.airbus.com`.

2. **Resolve contract** — Looks up the correct contract ID from the `otp-airbus` Kubernetes Secret (stored base64-encoded under the `contracts` key), selecting the appropriate contract by collection type: `PNEO` contracts for `airbus_pneo_data`, and `LEGACY` contracts for `airbus_phr_data` and `airbus_spot_data`.

3. **Resolve multi-acquisition items** — Detects whether a STAC item contains a `composed_of_acquisition_identifiers` property. If so, all constituent acquisition items are grouped into a single order. Items that are components of a multi-acquisition group are skipped when encountered directly.

4. **Submit order** — POSTs an order to `https://order.api.oneatlas.airbus.com/api/v1/orders`. The product type is derived from the collection ID (`PleiadesNeoArchiveMono/Multi`, `PleiadesArchiveMono`, or `SPOTArchive1.5Mono`). For bundles requesting a DEM or projection, additional options are resolved by querying the OneAtlas contract options endpoint. Returns a `salesOrderId` and a `customerReference` of the form `<workspace>_<timestamp>`.

5. **Publish "ordered" status** — Updates the STAC item with `order:status = ordered` and writes it to the workspace S3 bucket, then sends a Pulsar message on the `transformed` topic.

6. **Poll S3 for delivery** — Waits up to 24 hours for Airbus to deliver an archive matching `<customerReference>*<acquisitionId>.zip` in the commercial data bucket.

7. **Extract archive** — Downloads and extracts the `.zip` archive into a local directory named after the customer reference.

8. **Publish "succeeded" status** — Updates the STAC item with `order:status = succeeded`, rewrites asset `href` values to their S3 paths, and writes a local STAC catalog/collection/item bundle as the CWL output directory.

### SAR

The SAR pipeline follows the same structure with these differences:

- **In-progress check** — Before submitting, queries the Airbus SAR API (`sar.api.oneatlas.airbus.com/v1/sar/orders/*/items/status`) to check whether an order for the acquisition is already in a `submitted` state.
- **Submit order** — POSTs to `https://sar.api.oneatlas.airbus.com/v1/sar/orders/submit`. Order options are a flat structure of `productType`, `orbitType`, `gainAttenuation`, and optionally `mapProjection` and `resolutionVariant`.
- **Poll S3 for delivery** — Waits up to **7 days** (vs 24 hours for optical) to accommodate manual confirmation steps between Airbus and the customer. Archives are matched by prefix `SO_<orderId>` with suffix `.tar.gz`.
- **Extract archive** — Downloads and extracts `.tar.gz` archives.

On failure at any step, both adaptors write `order:status = failed` and an `order_failure_reason` string to the STAC record.

## STAC Order State Machine

State transitions use the [STAC Order extension](https://github.com/stac-extensions/order) fields (`order:status`, `order:id`, `order:date`):

```kroki-plantuml
@startuml
hide empty description

[*] --> orderable
orderable --> ordered
ordered --> succeeded
ordered --> failed

@enduml
```

## Optical Product Bundles

| Bundle | Processing Level | Bit Depth | Radiometric | Spectral |
|---|---|---|---|---|
| `Visual` | ortho | 8-bit | display | pansharpened natural colour |
| `General Use` | ortho | 12-bit | reflectance | pansharpened |
| `Analytic` | ortho | 12-bit | reflectance | bundle |
| `Basic` | primary | 12-bit | basic | bundle |

## SAR Product Options

| Option | Values |
|---|---|
| `product_type` | `SSC`, `MGD`, `GEC`, `EEC` |
| `orbit` | `rapid`, `science` |
| `resolution` | `RE`, `SE` |
| `projection` | `auto`, `UTM`, `UPS` |

## Configuration — Kubernetes Secrets

The adaptor reads two values from the Kubernetes Secret `otp-airbus` in the workspace namespace (`ws-<workspace>`):

| Key | Purpose |
|---|---|
| `otp` | Base64-encoded One-Time Pad key used to decrypt the Airbus API key stored in AWS Secrets Manager |
| `contracts` | Base64-encoded JSON object mapping contract IDs to their type strings (e.g. `{"<id>": "PNEO", "<id>": "LEGACY"}`) |

The encrypted API key is stored in AWS Secrets Manager under the secret ID `ws-<workspace>-<CLUSTER_PREFIX>`, keyed by `airbus`.

For detailed operator guidance including sanity-check commands, see [Data Adaptors — Airbus workspace secrets](data-adaptors.md#airbus-workspace-secrets-operator-reference).

## Key Dependencies

| Package | Role |
|---|---|
| `pystac` | STAC item parsing and serialisation |
| `boto3` | S3 read/write and AWS Secrets Manager access |
| `kubernetes` | Reading OTP keys and contract IDs from Kubernetes Secrets |
| `pulsar-client` | Pulsar producer for downstream event notification |
| `requests` | Airbus OneAtlas REST API calls |
