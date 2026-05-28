---
title: QGIS and XYZ Tile Integration
doc_status: ok
tags:
  - qgis
  - titiler
  - wms
  - layer-files
  - rc-ui
last_reviewed:
reviewed_by:
review_notes: Updated from eodhp-rc-ui and resource-catalog-support-utils source. WMS noted as not yet implemented.
---

# QGIS and XYZ Tile Integration

Covers how EODH data can be loaded into QGIS from the RC UI, how QLR layer definition files are generated, and the rendering configuration system behind it.

---

## Overview

Users can load EODH datasets into QGIS directly from the Integration Tools panel in the Data Finder. When a user selects a STAC item whose collection has a non-preview render configuration, a **QGIS** tab appears offering:

1. **Download QLR file** — generates a QGIS Layer Definition (`.qlr`) file pre-configured with the item's tile URL, ready to drag into QGIS.
2. **Copy XYZ tile URL** — copies the full TiTiler XYZ URL to the clipboard for manual use.

The active render (e.g. "Natural Color", "NDVI") is shown above both options. Some collections have multiple render options; the currently selected render is what gets baked into the QLR or copied URL.

---

## Which collections show the QGIS tab

The QGIS tab (and the Integration Tools panel as a whole) only appears if the collection has at least one non-preview render configuration. A render is considered preview-only if:

- Its title contains "preview" (case-insensitive), or
- It uses only the `thumbnail` asset, or
- It has a `quicklook_georeference` property.

Collections with only preview renders — currently `PSScene`, `SkySatCollect` — do not show the QGIS tab.

---

## Rendering configuration

Render configurations are bundled into the RC UI at build time (in `src/library/configFromServer.json`). Each collection entry defines one or more named renders, for example:

```json
"sentinel2_ard": {
  "renders": {
    "Natural Color": {
      "assets": ["cog"],
      "bidx": [3, 2, 1],
      "color_formula": "Gamma RGB 6 Saturation 0.8 Sigmoidal RGB 25 0.35",
      "nodata": 0
    },
    "NDVI": {
      "assets": ["cog"],
      "bidx": [7, 3],
      "expression": "(cog_b7-cog_b3)/(cog_b7+cog_b3)",
      "colormap_name": "rdylgn",
      "rescale": [[-1, 1]],
      "nodata": 0
    }
  }
}
```

Supported render properties that are passed through to TiTiler as query parameters: `bidx`, `rescale`, `colormap`, `colormap_name`, `color_formula`, `expression`, `variable`, `nodata`, `resampling`.

For **time-variable collections** (e.g. UKCP climate data), each STAC item carries properties that are matched against `eodhrc:render-key-properties` in the config, allowing each item to resolve to a different render automatically (e.g. a `var_id="pr"` item uses the Precipitation render).

---

## XYZ tile URL construction

The XYZ tile URL is built by the frontend from the active render config:

```
{VITE_TITILER_BASE_URL}/core/stac/tiles/WebMercatorQuad/{z}/{x}/{y}@1x
  ?url={encoded-stac-item-self-link}
  &bidx=3&bidx=2&bidx=1
  &rescale=0%2C200&rescale=0%2C200&rescale=0%2C200
  &color_formula=...
```

For multidimensional data (e.g. NetCDF), the path segment is `/xarray/tiles/{z}/{x}/{y}@1x` with a `variable` parameter added.

All render parameters are embedded in the URL. The tile service endpoint is `VITE_TITILER_BASE_URL` (must be set; no default).

---

## QLR file generation (resource-catalog-support-utils)

QLR files are generated server-side by the `resource-catalog-support-utils` FastAPI service.

### Endpoint

**`POST /api/v1/qlr`** (also available at `/qlr` for backward compatibility)

**Request:**
```json
{
  "item": { ... },       // full STAC Item object
  "xyzUrl": "https://titiler.../tiles/{z}/{x}/{y}?url=...&bidx=...",
  "renderId": "Natural Color"
}
```

`xyzUrl` must be HTTP/HTTPS and contain `{z}`, `{x}`, and `{y}` placeholders. `renderId` must be non-empty.

**Response:** `application/xml` with `Content-Disposition: attachment; filename="{item.id}.qlr"`

### What the QLR contains

The service uses a single generic XML template for all collections. It writes:

- **Layer name** — resolved from the STAC item's `properties.title` → `properties.name` → `id` (first non-null)
- **Data source** — `http-header:referer=&type=xyz&url={encoded-xyzUrl}` (QGIS XYZ tile source format)
- **Provider key** — `wms` (this is QGIS's internal name for its XYZ/tile provider)
- **CRS** — EPSG:3857 (Web Mercator)

The layer is set to visible by default. No collection-specific template logic exists — all collections use the same template.

### Architecture note

The service is stateless and rendering-agnostic. The frontend constructs the full TiTiler URL with all render parameters embedded and passes it in `xyzUrl`. The service only packages that URL into the QLR template — it does not call TiTiler or know about render configurations.

---

## Long URL problem (issue #61)

For datasets with embedded JSON colormaps (e.g. classified land cover), the XYZ URL can become extremely long:

```
/titiler/core/cog/tiles/{z}/{x}/{y}.png
  ?url=https://...
  &colormap=%7B%220%22%3A%5B0%2C0%2C0%2C0%5D%2C%221%22%3A...%7D
```

Long URLs can cause problems in QGIS's XYZ tile source, and may exceed proxy URL length limits.

The proposed fix was to switch to TiTiler's **WMS endpoint** for such layers, which would move the colormap to the server side and produce a short stable URL. **This is not yet implemented** — the current approach is still XYZ tiles with the full colormap in the URL.
