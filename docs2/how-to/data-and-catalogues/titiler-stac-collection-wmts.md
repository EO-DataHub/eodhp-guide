---
title: Adding STAC Collection to TiTiler WMTS using the Renders Extension
doc_status: ok
last_reviewed:
reviewed_by:
review_notes: Migrated from docs/operations/titiler/stac-collection-wmts.md.
---
# Adding STAC Collection to TiTiler WMTS using the Renders Extension

## Purpose

TiTiler STAC can expose STAC Collections as Web Map Tile Service (WMTS) layers so users can integrate raster data into GIS tools (for example QGIS) or web mapping libraries, with mosaiced tiles served dynamically from STAC items.

## When to Use

Use this guide when adding or updating a STAC Collection in EO DataHub that you want to expose through the TiTiler STAC API as a WMTS-compatible raster mosaic.

## Operation

### Step 1: Prepare the STAC Collection

Ensure your STAC Collection JSON has a `renders` section. This tells TiTiler how to mosaic and display your raster data.

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

**Key parameters:**

- **assets**: Which STAC assets to include (usually Cloud Optimised GeoTIFFs).
- **bidx**: Bands to map to RGB.
- **rescale**: Input value ranges used when mapping to display colours.
- **resampling**: Raster resampling (`nearest` or `bilinear`, etc.).
- **tilematrixsets**: Zoom levels TiTiler should generate tiles for.

### Step 2: Deploy or Update your STAC Collection

Update the collection JSON in the EO DataHub STAC catalogue, including the `renders` block.

1. Change the collection payload (for example by editing the defaults used by **harvest-transformer** in [`render_processor.py`](https://github.com/EO-DataHub/harvest-transformer/blob/main/harvest_transformer/render_processor.py) — line numbers move over time; inspect the file on `main` for the active `renders` wiring).
2. Commit, merge, and run **harvest-transformer** as described in that repository’s README or release process.

### Step 3: Test WMTS Access

Confirm the WMTS layer by requesting **GetCapabilities**:

```
https://eodatahub.org.uk/api/catalogue/stac/catalogs/<catalog-path>/wmts?request=GetCapabilities&service=WMTS
```

Example:

```
https://eodatahub.org.uk/api/catalogue/stac/catalogs/public/catalogs/ceda-stac-catalogue/wmts?request=GetCapabilities&service=WMTS
```

The response is XML listing layers. Check that your layer appears as expected.

### Step 4: Add the Layer to GIS Software (e.g., QGIS)

- In QGIS, add a **WMTS** layer.
- Paste the WMTS base URL from Step 3.
- Pick the layer entry to display it.

## Requirements

- Ability to change STAC Collections in the EO DataHub catalogue pipeline.
- Basic familiarity with STAC JSON.

## Useful Information

- TiTiler STAC supports several raster formats; **COGs are recommended**. The current TiTiler STAC deployment does **not** support multidimensional (xarray-style) sources.
- COG assets must be reachable by TiTiler (public object URLs or credentials the service can use).
- You can define multiple `renders` per collection (for example false-colour composites).

**Related:** Catalogue ingest walkthrough [Sample data ingest](sample-data-ingest.md); [harvest-transformer](../../reference/repositories.md#harvest-transformer) in [Repositories](../../reference/repositories.md).
