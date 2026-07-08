---
title: Commercial Data Purchasing Pipeline
doc_status: unreviewed
last_reviewed:
reviewed_by:
review_notes:
tags:
  - commercial-data
  - workflows
  - stac
---
# Commercial Data Purchasing Pipeline

Explains how a commercial data purchase flows from the user interface through to delivery of data assets in a workspace.

## Related Repositories

- [RC UI](https://github.com/EO-DataHub/eodhp-rc-ui)
- [Purchase API](https://github.com/EO-DataHub/resource-catalogue-fastapi)
- [Ordering workflows / data adaptors](https://github.com/EO-DataHub/commercial-data-adaptors)

## RC UI

1. Authenticated user finds a STAC item to purchase.
2. User selects the workspace from which to purchase data (e.g. `sparkgeouser`).
3. Front end presents purchase options (bundles, licence, etc.).
4. Purchase API is called to retrieve a quote.
5. After receipt of quote, Purchase API is called again to execute the order.

## Purchase API

The Purchase API orchestrates the creation of the order-tracking STAC item and triggers the appropriate adaptor workflow:

- Calls the ordering workflow in the data provider's workspace (e.g. a workflow in the `airbus` workspace).
- Validates licence type (Airbus optical/radar), product bundle (Planet or Airbus), and radar options (for Airbus SAR).
- Extracts the user workspace from the JWT.
- Verifies an API key exists for the provider (Planet/Airbus); for Airbus, validates the contract ID.
- Fetches the original STAC item from the commercial catalogue.
- Creates a tagged item ID (includes product bundle, radar options, and a coordinates hash).
- Updates the item with:
    - Order status: `pending`
    - Order options (product bundle, coordinates, end user, licence, radar options)
    - Intersected geometry (if AOI coordinates provided)
    - Timestamps (created/updated)
- Uploads catalogue, collection, and item to S3 (both workspace and transformed paths).
- Selects the adaptor:
    - `airbus-sar-adaptor` for Airbus SAR
    - `airbus-optical-adaptor` for Airbus optical
    - `planet-adaptor` for Planet
- **On failure:** updates STAC item status to `failed`, uploads the failed item to S3, sends a Pulsar message to topic `transformed` for catalogue update, and returns a 500 error.
- **On success:** sends a Pulsar message to topic `transformed` to update the catalogue (includes workspace, bucket name, and STAC item keys), and returns 201 with the created STAC item and `Location` header.

## Ordering Workflow (Data Adaptor)

The ordering workflow executes in the data provider workspace (e.g. `airbus`):

- Fetches two API keys:
    - **Key 1:** from the purchasing user's workspace (e.g. `sparkgeouser`) for their data provider account.
    - **Key 2:** the global EODH data provider account key (e.g. `planet` or `airbus`) used for things like querying the territory list.
- The STAC item was already created by the Purchase API as a static S3 JSON and is harvested into the Elasticsearch Resource Catalogue.
- Uses STAC Order extension fields (`order:status`, `order:id`, `order:date`) to track status progression: `orderable` → `ordered` → `succeeded` or `failed`.
- Calls the data provider's API (e.g. Airbus One Atlas API), passing in the API keys and order details.
- Monitors an EODH S3 bucket in which the data provider deposits the purchase:
    - File formats: `.zip` (Airbus Optical), `.tar.gz` (Airbus SAR, Planet)
    - Timeouts: 24 hours (default), 7 days for SAR
- Unzips/extracts the delivered file.
- Moves files into the user workspace (e.g. `sparkgeouser`).
- Updates the STAC item in the user workspace to change order status and add asset records for delivered files. Assets identified vary by provider: primaryAsset, quicklook, thumbnail, metadata, masks.

## Execution Model: public vs user-service

Whether a deployed ordering workflow actually runs **in the provider's workspace** (with access to that workspace's stored provider credentials) or **in the calling user's own workspace** depends entirely on the workflow's access policy — not on anything in the Purchase API.

ADES checks the workflow's access-policy.json (see the [access-policy schema reference](../../reference/services/workflow-runner.md#access-policies-public-vs-user-service)) in this order:

| | Executes in | Gets access to |
|---|---|---|
| `"public": true` | Calling workspace (the ordering user's own namespace) | Only the calling workspace's storage/secrets |
| `"user_service": true` | Deploying/owning workspace (e.g. `ws-planet`, `ws-airbus`) | Additional access to the owning workspace's storage, secrets, block store |

Commercial data adaptors must be deployed as `user_service` workflows: without provider-workspace access, the adaptor can't read the provider API keys / cloud credentials linked to that provider's workspace (e.g. the Airbus `otp-airbus` secret), since those are only ever stored against the provider workspace, not the purchasing user's.

The API surfaces which mode a deployed workflow is actually running in via an `AccessReasoning` field on process listings (`"Workflow is a user-service"` / `"Workflow is public"`) — check this field first when a purchase fails with missing-credential errors, since it usually means the workflow was deployed as `public` instead of `user_service`.

## COG Conversion (Planned)

An additional workflow is planned to convert delivered assets to Cloud-Optimised GeoTIFF (COG):

- Uses the XML file asset (dmap) for mosaic definition.
- Converts files into COG format.
- Adds an additional asset to the STAC item.

## See Also

- [Data Adaptors reference](../../reference/services/data-adaptors.md) — service configuration, secrets, deployment
- [Testing commercial data adaptor changes](../../how-to/data-and-catalogues/test-commercial-data-adaptor.md)
