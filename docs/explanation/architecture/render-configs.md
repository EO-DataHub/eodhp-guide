---
title: RC UI Render Configurations
doc_status: ok
tags:
  - rc-ui
  - titiler
  - stac
last_reviewed:
reviewed_by:
review_notes: Updated from eodhp-rc-ui source. Originally drafted from issues #85, #115, #128, #163. Issue #42 (future enhanced viz) also referenced.
---

# RC UI Render Configurations

Explains how the Resource Catalogue UI controls rendering for each collection, where the configuration lives, and the known data quality issues affecting CMIP6 rendering.

---

## How render configs work

The RC UI holds a per-collection rendering configuration in:

```
eodhp-rc-ui/src/library/configFromServer.json
```

This file defines — for each STAC collection — which assets to render, which TiTiler endpoint to use, band indices, rescaling parameters, colormaps, and whether thumbnails or quicklooks are available. The UI reads this at runtime to construct TiTiler tile requests.

Because rendering parameters are defined here rather than in the STAC catalogue, updating a render config requires a frontend code change rather than a catalogue update.

### Adding or updating a render config

1. Identify the STAC collection name and the asset key(s) to render.
2. Determine the TiTiler parameters: band indices, rescaling range, colormap (if any), endpoint type.
3. Check whether a thumbnail or quicklook asset exists in the STAC item — these can be displayed while COG conversion is pending.
4. Add or update the entry in `configFromServer.json`.
5. Test against a sample item in the test environment before promoting.

---

## Render parameter reference

| Parameter | Usage | Example |
|-----------|-------|---------|
| `assets` | Asset keys to read from the STAC item | `["cog"]`, `["reference_file"]` |
| `bidx` | Band indices (1-based) | `[3,2,1]` for RGB, `[7,3]` for two-band expression |
| `rescale` | Per-band min/max stretch | `[[0,200],[0,200],[0,200]]` |
| `nodata` | NoData value to mask | `0`, `-9999` |
| `colormap_name` | Named matplotlib colormap | `"viridis"`, `"rdylgn"`, `"turbo"`, `"coolwarm"` |
| `colormap` | Custom discrete colormap (value → hex) | `{"0": "#000000", "1": "#ffffff"}` |
| `color_formula` | Band processing expression | `"Gamma RGB 6 Saturation 0.8 Sigmoidal RGB 25 0.35"` |
| `expression` | Algebraic band index | `"(cog_b7-cog_b3)/(cog_b7+cog_b3)"` |
| `variable` | xarray variable name (netCDF/zarr) | `"pr"`, `"tas"`, `"analysed_sst"` |
| `auth` | Whether tile requests require authentication | `true`, `false` |
| `quicklook_georeference` | Display as georeferenced image rather than tiles | `"bbox-squared"`, `"geometry"` |
| `thumbnail_asset` | Asset key to use for thumbnail fallback | `"thumbnail"`, `"external_thumbnail"` |
| `thumbnail_auth` | Whether thumbnail requires authentication | `true`, `false` |

### Render type categories

| Type | Key parameters | Examples |
|------|---------------|----------|
| RGB / multispectral | `bidx` + `rescale` | Sentinel-2 Natural Color, Airbus PHR |
| Single-band + colormap | `bidx` + `colormap_name` + `rescale` | NDVI, SST, aerosol |
| Discrete classification | `colormap` (value→colour map) | Land cover, landcover-UK |
| Algebraic index | `expression` + `bidx` | NDVI on Sentinel-2 |
| Quicklook / preview | `quicklook_georeference` | Planet PSScene, Airbus SPOT |
| NetCDF / xarray | `variable` (routes to titiler-multidim) | UKCP, CMIP6, EOCIS SST/LST/aerosol |

### TiTiler endpoints

The config loader routes each render to one of two TiTiler backends:

- **Standard STAC** — `/core/stac/tiles/WebMercatorQuad/{z}/{x}/{y}@1x` — used for COG and most raster assets.
- **Multidimensional** — `/xarray/tiles/{z}/{x}/{y}@1x` — used when the render has a `variable` field (netCDF/zarr via kerchunk reference files).

---

## User collections

Commercial data purchased by a user is stored under a `my-{base_collection}` collection ID. The base collection (e.g. `PSScene`) holds public previews only; the user collection (e.g. `my-PSScene`) provides authenticated access to the full COG assets.

`auth: true` on a render means tile requests are sent with the user's workspace token. `thumbnail_auth` controls whether the thumbnail image itself requires authentication.

---

## Time-variable collections

Collections marked with `"eodhrc:time-variable-items": true` (currently UKCP and CMIP6) can have each STAC item automatically resolve to a different render based on item properties. The `eodhrc:render-key-properties` array specifies which STAC item property values are matched against render IDs:

| Collection | Render key properties |
|------------|----------------------|
| `ukcp` | `["var_id"]` |
| `cmip6` | `["cmip6:variable_id", "cmip6:frequency", "cmip6:experiment_id"]` |

---

## Future: enhanced visualisation (issue #42)

- **Time slice UI** — a slider component for temporal datasets (CMIP6, EOCIS time-series).
- **Render extension format** — attaching rendering hints directly to STAC items to reduce dependency on `configFromServer.json`.
- **Virtual raster tile** — for datasets not renderable via standard TiTiler pathways.
- **Rotated pole coordinate systems** — some climate datasets use non-standard projections requiring TiTiler changes (a TPZ dependency).
