# Recap of how the purchasing workflow works

## Related Repositories
* [RC UI](https://github.com/EO-DataHub/eodhp-rc-ui)
* [Purchase API](https://github.com/EO-DataHub/resource-catalogue-fastapi)
* [Ordering workflows](https://github.com/EO-DataHub/commercial-data-adaptors)

## RC UI
* Authenticated user finds STAC item to purchase
* User chooses workspace from which to purchase data (e.g. `sparkgeouser`)
* Front end presents purchase options (bundles, license, etc.)
* Purchase API called to retrieve quote
* After receipt of quote, Purchase API called to execute order

## Purchase API
* Calls the ordering workflow in the data provider's workspace (e.g. workflow in `airbus` workspace)
* Validates licence type (Airbus optical/radar)
* Validates product bundle (Planet or Airbus)
* Validates radar options (for Airbus SAR)
* Extracts user workspace from JWT
* Verifies API key exists for the provider (Planet/Airbus)
* For Airbus: validates contract ID
* Fetches the original STAC item from the commercial catalog
* Creates a tagged item ID (includes product bundle, radar options, coordinates hash)
* Updates item with:
  - Order status: pending
  - Order options (product bundle, coordinates, end user, licence, radar options)
  - Intersected geometry (if AOI coordinates provided)
  - Timestamps (created/updated)
* Uploads catalog, collection, and item to S3 (both workspace and transformed paths)
* Selects adaptor:
  - `airbus-sar-adaptor` for Airbus SAR
  - `airbus-optical-adaptor` for Airbus optical
  - `planet-adaptor` for Planet
* On failure:
  - Updates STAC item status to failed
  - Uploads failed item to S3
  - Sends Pulsar message to topic `transformed` for catalog update
  - Returns 500 error
* On success:
  - Sends Pulsar message to topic `transformed` to update the catalog (includes workspace, bucket name, and STAC item keys)
  - Returns 201 with the created STAC item and Location header

## Ordering workflow
* Is executed in data provider (e.g. `airbus`) workspace
* Fetches 2 API keys:
  - Key 1: from user (e.g. `sparkgeouser`) workspace for their data provider account
  - Key 2: the global EODH data provider account key (i.e. `planet` or `airbus`) used for things like querying territory list
* The STAC item was already created by the Purchase API (as static S3 JSON) and is harvested into the Elasticsearch Resource Catalogue
* Uses STAC Order extension fields (`order:status`, `order:id`, `order:date`) to track status progression: `orderable` → `ordered` → `succeeded` or `failed`
* Calls the data provider's API (e.g. Airbus One Atlas API), passing in the API keys and order details
* Monitors an EODH S3 bucket in which the data provider deposits the purchase:
  - File formats: `.zip` (Airbus Optical), `.tar.gz` (Airbus SAR, Planet)
  - Timeouts: 24 hours default, 7 days for SAR
* Unzips/extracts the delivered file
* Moves the files into the user (e.g. `sparkgeouser`) workspace
* Updates the STAC item in the user workspace to change order status and add asset records for delivered files:
  - Assets identified: primaryAsset, quicklook, thumbnail, metadata, masks (varies by provider)

# Changes for COG conversion
* Additional workflow to convert assets to COGS
* Uses the XML file asset (dmap?) for definition of mosaic
* Converts the files into COG
* Adds an additional asset to the STAC item