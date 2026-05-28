---
title: Open Cosmos Adaptor
doc_status: unreviewed
tags:
  - commercial-data
  - stac
  - workflows
last_reviewed:
reviewed_by:
review_notes:
---
# Open Cosmos Adaptor

Reference for the Open Cosmos commercial data adaptor, which orders satellite imagery from the Open Cosmos DataCosmos API and delivers it into an EO DataHub workspace. It runs as a short-lived Kubernetes pod orchestrated by a CWL (Common Workflow Language) workflow, and terminates after completing or failing each order batch.

**How-to:** [Test commercial data adaptor changes](../../how-to/data-and-catalogues/test-commercial-data-adaptor.md) · **Explanation:** [Commercial Data Purchasing Pipeline](../../explanation/architecture/commercial-data-purchasing.md)

## Execution Model

The CWL workflow (`open-cosmos.cwl`) wraps a single `CommandLineTool` step that pulls a Docker image and invokes `python -m open_cosmos_adaptor`. The CWL runner passes positional arguments and injects `CLUSTER_PREFIX` as an environment variable. The tool's output is the working directory (`.`), which contains the STAC records written during execution.

- **Container image:** `public.ecr.aws/eodh/open-cosmos-adaptor:0.1.0`
- **Source code:** [EO-DataHub/commercial-data-adaptors](https://github.com/EO-DataHub/commercial-data-adaptors)

## Inputs

| Parameter | Description |
|---|---|
| `workspace` | Workspace identifier within the platform |
| `workspace_bucket` | S3 bucket where workspace STAC records are stored |
| `commercial_data_bucket` | S3 bucket used to receive commercial data |
| `pulsar_url` | Apache Pulsar broker URL for downstream event messaging |
| `cluster_prefix` | Platform environment prefix (injected as `CLUSTER_PREFIX`) |
| `stac_key` | Directory containing one or more STAC catalogs describing items to order |
| `coordinates` | Area of interest coordinates (passed through; not currently consumed by the Python) |

## Processing Pipeline

For each STAC item found in the input catalogs, the adaptor executes these steps in sequence, aborting the item and writing a failure record on any error:

1. **Authenticate** — Obtains an OAuth2 bearer token via the client credentials flow against `login.open-cosmos.com`, using `CLIENT_ID` and `CLIENT_SECRET` sourced from Kubernetes secrets. The token is cached in-process.

2. **Resolve contract** — Fetches the organisation's data-access policies from the Open Cosmos DPAP API to identify the default contract ID. `ORGANIZATION_ID` is also sourced from Kubernetes secrets.

3. **Submit order** — POSTs an `IMAGE` order to the Open Cosmos orders API, specifying the collection ID, item ID, and processing level from the STAC item's `processing:level` property. The order must return a `PAID` status to proceed.

4. **Publish "ordered" status** — Updates the STAC item with `order:status = ordered` and `order:id`, writes it to two S3 paths in the workspace bucket (a raw path and a `transformed/catalogs/…` path), then sends a Pulsar message on the `transformed` topic to notify downstream catalog ingestion services.

5. **Download assets** — Streams each asset file from the Open Cosmos API to a local directory named after the order ID, using the bearer token for authentication.

6. **Upload to S3** — Uploads the downloaded asset files to the workspace S3 bucket via boto3.

7. **Publish "succeeded" status** — Updates the STAC item with `order:status = succeeded`, rewrites asset `href` values to their S3 paths, and writes a local STAC catalog/collection/item bundle as the CWL output directory.

On failure at any step, the adaptor writes `order:status = failed` and an `order_failure_reason` string to the STAC record before exiting.

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

## Configuration — Kubernetes Secrets

Three values must be present as environment variables, typically mounted from a Kubernetes Secret:

| Variable | Purpose |
|---|---|
| `CLIENT_ID` | Open Cosmos OAuth2 client ID |
| `CLIENT_SECRET` | Open Cosmos OAuth2 client secret |
| `ORGANIZATION_ID` | Open Cosmos organisation identifier (used to resolve the default contract) |

## Key Dependencies

| Package | Role |
|---|---|
| `pystac` | STAC item parsing and serialisation |
| `boto3` | S3 read/write |
| `pulsar-client` | Pulsar producer for downstream event notification |
| `requests` | Open Cosmos REST API calls |
