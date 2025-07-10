# Adding STAC Collection to TiTiler WMTS using the Renders Extension

## Purpose

TiTiler STAC can expose STAC Collections as Web Map Tile Service (WMTS) layers. This allows users to easily integrate raster data into GIS tools like QGIS or web mapping libraries serving mosaiced tiles dynamically from STAC items.

## When to Use

Use this guide when adding or updating a STAC Collection in EO DataHub that you want to expose through the TiTiler STAC API as a WMTS-compatible raster mosaic.

## Operation

### Step 1: Prepare the STAC Collection

Ensure your STAC Collection JSON has a `renders` section. This tells TiTiler exactly how it should mosaic and display your raster data.

A typical `renders` configuration looks like this:

```json
{
    "renders": {
        "rgb": {
            "title": "RGB",
            "assets": ["cog"],
            "bidx": [1, 2, 3],
            "rescale": [
                [0, 100],
                [0, 100],
                [0, 100]
            ],
            "resampling": "nearest",
            "tilematrixsets": {
                "WebMercatorQuad": [0, 30]
            }
        }
    }
}
```

**Key Parameters Explained:**

- **assets**: Which STAC assets to include (usually Cloud Optimised GeoTIFFs).
- **bidx**: The bands you want to map to RGB
- **rescale**: The value ranges for the raster data to ensure correct colors
- **resampling**: How to handle raster resampling (typically `nearest` or `bilinear`).
- **tilematrixsets**: The zoom levels you want TiTiler to generate tiles for.

### Step 2: Deploy or Update your STAC Collection

Update the collection JSON in the EO DataHub's STAC catalogue, making sure the `renders` block is included.

1. Modifying the STAC collection JSON file which you can do through the DataHub's `harvest-transformer` by updating the object here: https://github.com/EO-DataHub/harvest-transformer/blob/225947002216ea5543296468dac143a91c5e4030/harvest_transformer/render_processor.py#L4
2. Committing changes and merging them in and running the `harvest-transformer` (see repository documentation for details on how to do this).

### Step 3: Test WMTS Access

After updating your STAC Collection, confirm the WMTS layer is correctly configured by requesting the WMTS GetCapabilities document:

```
https://eodatahub.org.uk/api/catalogue/stac/catalogs/<catalog-path>/wmts?request=GetCapabilities&service=WMTS
```

For example:

```
https://eodatahub.org.uk/api/catalogue/stac/catalogs/public/catalogs/ceda-stac-catalogue/wmts?request=GetCapabilities&service=WMTS
```

This returns an XML document describing available layers. Ensure your new layer appears correctly here.

### Step 4: Adding the Layer to GIS software (e.g., QGIS)

- In QGIS, add a new WMTS layer.
- Paste the WMTS URL obtained in Step 3.
- Select the desired layer to visualise it.

## Requirements

- Access to edit STAC Collections in the EO DataHub catalogue repository.
- Basic familiarity with STAC JSON configuration.

## Useful Information

- TiTiler STAC supports various raster formats, but Cloud Optimised GeoTIFFs (COGs) are recommended. There is not support for multidimensional data in the current TiTiler STAC implementation.
- Ensure all COG assets are publicly accessible or that TiTiler has appropriate access rights.
- Multiple renders can be configured per collection, allowing users to visualise data in different ways (e.g. false-color composites).
