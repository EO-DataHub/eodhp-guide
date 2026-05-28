---
title: TiTiler Private Data Rendering
doc_status: needs-verification
tags:
  - titiler
  - stac
  - commercial-data
last_reviewed:
reviewed_by:
review_notes: Updated from titiler, titiler-stacapi, and eodhp-rc-ui source. Originally drafted from issues #5, #75, #107, #131.
---

# TiTiler Private Data Rendering

Explains how TiTiler renders private workspace data (commercial purchases, user-uploaded COGs) for authenticated users, and how authentication is handled across the STAC and COG tile endpoints.

---

## Two TiTiler deployments

There are two separate TiTiler deployments in the EODH cluster:

| Deployment | Purpose |
|------------|---------|
| `titiler` | Main deployment. Handles COG and STAC endpoints for the RC UI. Includes EODH-specific auth and private S3 access. |
| `titiler-stacapi` | Dynamic tile generation from the EODH STAC catalogue. Queries the catalogue API to discover items and generate tiles or mosaics on demand. Exposes a WMTS endpoint and supports tile generation per-item (`/collections/{id}/items/{item_id}/tiles/...`) and per-collection mosaic (`/collections/{id}/tiles/...`). Used for catalogue-driven tiling rather than direct asset URLs. |

---

## Two rendering modes (main titiler)

TiTiler serves tiles via two distinct endpoints:

| Endpoint | URL pattern | Input | Use case |
|----------|-------------|-------|----------|
| COG endpoint | `/titiler/core/cog/tiles/...` | Direct asset URL (HTTPS or S3) | Rendering a specific file |
| STAC endpoint | `/titiler/core/stac/tiles/...` | STAC item URL + asset name | Rendering via catalogue metadata |

The RC UI uses the **STAC endpoint** for consistency with how it handles public data. This requires TiTiler to first fetch the STAC item JSON to resolve the asset's S3 location, then read the raster.

---

## Authentication for private data

Private workspace data lives in `s3://workspaces-eodhp/<workspace>/...`. TiTiler has two AWS IAM roles, configured via environment variables:

| Env var | Used for |
|---------|----------|
| `AWS_PRIVATE_ROLE_ARN` | Reading objects from private workspace S3 buckets |
| `AWS_PUBLIC_ROLE_ARN` | Public data and public workspace files |

When a user makes a tile request for private data, they include a bearer token:

```
Authorization: Bearer <user-token>
```

TiTiler uses this token to assume the private S3 role via `AssumeRoleWithWebIdentity`, obtaining temporary AWS credentials scoped to that user's access.

### Auth decision flow

`auth.py` applies one of five scenarios depending on the URL and token:

1. **Workspace private file + auth header** → assume `AWS_PRIVATE_ROLE_ARN`
2. **EODH STAC item URL + auth header** → assume `AWS_PRIVATE_ROLE_ARN`
3. **Public workspace file + no auth header** → assume `AWS_PUBLIC_ROLE_ARN`
4. **Generic S3/HTTPS URL** → pass through as-is
5. **EFS local path** → JWT validation of workspace access

---

## STAC endpoint authentication implementation

The STAC endpoint (`/stac/tiles/...`) requires TiTiler to first fetch the STAC item JSON to resolve asset locations, then read the raster. This means the Authorization header must be forwarded at three levels: the STAC item fetch, the asset href resolution, and the rasterio S3 read.

### Forwarding the Authorization header to `STACReader`

When the STAC reader is used, the Authorization header from the incoming tile request is forwarded to the STAC item fetch:

```python
if reader == STACReader:
    reader = RewriteSTACReader
    auth_header = request.headers.get("Authorization")
    if auth_header:
        extra_kwargs["fetch_options"] = {"headers": {"Authorization": auth_header}}
```

The COG endpoint (`/cog/tiles`) does not need this — it reads the asset URL directly without a STAC metadata fetch.

### `RewriteSTACReader` — URL rewriting for asset hrefs

STAC item assets reference workspace files as HTTPS URLs (e.g. `https://<workspace>.eodatahub-workspaces.org.uk/files/...`). A `RewriteSTACReader` subclass rewrites these to `s3://` URIs for direct S3 access:

```python
class RewriteSTACReader(STACReader):
    def _get_asset_info(self, asset: str):
        info = super()._get_asset_info(asset)
        resolved_path, _ = rewrite_https_to_s3_if_needed(info["url"])
        info["url"] = resolved_path
        return info
```

### Propagating AWS credentials to per-asset readers

`STACReader` reads each asset in a thread pool where each asset gets its own `rasterio.Env`. The private-role AWS credentials are passed explicitly via the `ctx` parameter so each thread has access to them:

```python
with rasterio.Env(**updated_env) as rasterio_env:
    if reader_cls == RewriteSTACReader:
        extra_kwargs["ctx"] = lambda: rasterio_env
```

---

## RC UI tile authentication

The RC UI passes the user's bearer token in tile requests using MapLibre's native **`transformRequest` callback**. When a raster layer mounts, it registers the tile base URL as requiring authentication. The `transformRequest` callback checks each tile request against this registry and injects `Authorization: Bearer <token>` when needed.

Two token types are used:
- **Keycloak token** — for Keycloak-protected collections
- **Workspace token** — for user/purchased data collections (fetched from `POST /api/workspaces/{workspace}/me/sessions`)

The `auth: true` flag in a render config entry triggers this flow. Renders without `auth` default to requiring auth for user collections and not requiring it for public collections. See [RC UI Render Configurations](render-configs.md) for the full render parameter reference.

---

## Known tech debt

### Direct modifications to `factory.py`

EODH's authentication customisations are made directly inside `titiler/core/factory.py` — `resolve_src_path_and_credentials()` is called at ~20 locations throughout the file. This makes it difficult to merge upstream TiTiler updates. The recommended approach is to use TiTiler's dependency injection system to keep EODH-specific code in a separate file. This refactor is planned for a future phase.

### Bitnami base image deprecated (unresolved)

The TiTiler Docker image uses `bitnami/python` as its base image, which has been deprecated. A temporary workaround is in place:

```dockerfile
FROM bitnamilegacy/python:${PYTHON_VERSION}
```

This is still the current state. A permanent migration to an alternative base image (e.g. `python:3.12-slim`) is needed.

---

## Testing private rendering

To verify the STAC endpoint works for private data:

```bash
# Should return a tile image (not 401/403/500)
curl --request GET \
  --url 'https://eodatahub.org.uk/titiler/core/stac/tiles/WebMercatorQuad/12/2051/1321@1x
    ?url=https://eodatahub.org.uk/api/catalogue/stac/catalogs/user/catalogs/<workspace>/...
    &assets=<asset-name>' \
  --header "Authorization: Bearer $TOKEN"
```

**Important:** The asset referenced must be a valid COG. Airbus orders are delivered as regular GeoTIFFs (not COG). These will fail with `not recognized as a supported file format` until the COG conversion workflow has processed them and added a `cog` asset to the STAC item. Use Planet SkySat or PlanetScope sample data (which are already COGs) as a reliable test target.
