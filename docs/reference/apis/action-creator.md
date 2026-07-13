---
title: 02. Action Creator API
doc_status: ok
last_reviewed: 2026-06-01
reviewed_by: geodowd
review_notes: "Written from live OpenAPI spec at staging.lot2.eodatahub.org.uk/api/v1.2/openapi.json"
tags:
  - api
  - action-creator
---
# 02. Action Creator API

The Action Creator API is a separate REST API (not rooted under the main EODH `/api/catalogue` or `/api/workspaces` hierarchy) for submitting and managing geospatial analysis workflows. It is served from its own versioned base path.

**Base URL:** `https://staging.lot2.eodatahub.org.uk/api/v1.2`

The API is versioned independently. `/api/v1.2` is the current documented version.

## Authentication

All Action Creator endpoints (except `/health/ping`) require a Bearer token in the `Authorization` header.

### Obtain a token

```
POST /auth/token
```

Request body:

```json
{
  "username": "your-username",
  "password": "your-password"
}
```

Response (`200`):

```json
{
  "access_token": "...",
  "expires_in": 300,
  "refresh_token": "...",
  "refresh_expires_in": 1800,
  "token_type": "Bearer",
  "session_state": "...",
  "scope": "..."
}
```

### Introspect a token

```
POST /auth/token/introspection
Authorization: Bearer <token>
```

Returns token claims including `active`, `workspaces`, `preferred_username`, `exp`, and Keycloak realm/resource roles.

## Endpoints

| Path | Method | Description |
| ---- | ------ | ----------- |
| `/health/ping` | GET | Health check — returns `"pong"` |
| `/auth/token` | POST | Obtain a Bearer token |
| `/auth/token/introspection` | POST | Introspect a token |
| `/action-creator/functions` | GET | List available workflow functions |
| `/action-creator/presets` | GET | List available workflow presets |
| `/action-creator/submissions` | POST | Submit a workflow |
| `/action-creator/submissions` | GET | List job history |
| `/action-creator/submissions` | DELETE | Batch cancel or delete jobs |
| `/action-creator/submissions/{submission_id}` | GET | Get status of a single job |
| `/action-creator/submissions/{submission_id}` | DELETE | Cancel or delete a single job |

## Functions

```
GET /action-creator/functions
Authorization: Bearer <token>
```

Optional query parameter: `?collection=<stac-collection-id>` — filters to functions compatible with that STAC collection.

Returns a list of `ActionCreatorFunctionSpec` objects describing each available function, its inputs, and outputs. The following function identifiers are available:

| Identifier | Name |
| ---------- | ---- |
| `ndvi` | Normalised Difference Vegetation Index |
| `ndwi` | Normalised Difference Water Index |
| `nbr` | Normalised Burn Ratio |
| `savi` | Soil-Adjusted Vegetation Index |
| `evi` | Enhanced Vegetation Index |
| `cya_cells` | Cyanobacteria Cells |
| `doc` | Dissolved Organic Carbon |
| `cdom` | Coloured Dissolved Organic Matter |
| `turb` | Turbidity |
| `clip` | Clip outputs to an AOI |
| `land-cover-change-detection` | Land Cover Change Detection |
| `water-quality` | Water Quality |

Most spectral-index functions share the same inputs:

| Input | Type | Required | Notes |
| ----- | ---- | -------- | ----- |
| `aoi` | GeoJSON Polygon | No | Area of interest |
| `date_start` | datetime (ISO 8601) | No | Start of temporal range |
| `date_end` | datetime (ISO 8601) | No | End of temporal range |
| `stac_collection` | string | No | STAC collection ID |
| `limit` | integer (1–10000) | No | Max items to process; default 1000 |

`clip` only takes `aoi`. `land-cover-change-detection` takes `aoi`, `date_start`, `date_end`, and `stac_collection` (no `limit`).

## Presets

```
GET /action-creator/presets
Authorization: Bearer <token>
```

Returns a list of pre-configured workflow specifications ready to submit. Each preset has an `identifier`, `name`, `description`, and a `workflow` object (same shape as the submission request).

## Submitting a Workflow

```
POST /action-creator/submissions
Authorization: Bearer <token>
Content-Type: application/json
```

### Request body

```json
{
  "workspace": "my-workspace",
  "workflow": {
    "<step-name>": {
      "identifier": "<function-identifier>",
      "order": 0,
      "inputs": { ... }
    }
  }
}
```

The `workflow` object is a map of arbitrary step names to step definitions. `order` controls execution sequence — steps with lower order values run first. Steps can be chained (e.g., run `ndvi` at order 0, then `clip` at order 1).

### Example — NDVI with clipping

```json
{
  "workspace": "my-workspace",
  "workflow": {
    "ndvi": {
      "identifier": "ndvi",
      "order": 0,
      "inputs": {
        "stac_collection": "sentinel-2-l2a-ard",
        "date_start": "2024-01-01T00:00:00Z",
        "date_end": "2024-12-31T23:59:59Z",
        "aoi": {
          "type": "Polygon",
          "coordinates": [[
            [-0.512, 51.446], [-0.512, 51.497],
            [-0.409, 51.497], [-0.409, 51.446],
            [-0.512, 51.446]
          ]]
        }
      }
    },
    "clip": {
      "identifier": "clip",
      "order": 1,
      "inputs": {
        "aoi": {
          "type": "Polygon",
          "coordinates": [[
            [-0.512, 51.446], [-0.512, 51.497],
            [-0.409, 51.497], [-0.409, 51.446],
            [-0.512, 51.446]
          ]]
        }
      }
    }
  }
}
```

### Response (`202 Accepted`)

Returns an `ActionCreatorJob` with a `submission_id` (UUID), the submitted spec, `status: "submitted"`, and `submitted_at`.

## Job History

```
GET /action-creator/submissions
Authorization: Bearer <token>
```

Query parameters:

| Parameter | Type | Default | Description |
| --------- | ---- | ------- | ----------- |
| `workspace` | string | — | Filter to a specific workspace |
| `status` | array of status values | — | Filter by one or more job statuses |
| `order_by` | string | `submitted_at` | Field to sort by: `submission_id`, `status`, `function_identifier`, `submitted_at`, `finished_at`, `successful` |
| `order_direction` | `asc` \| `desc` | `asc` | Sort direction |
| `page` | integer ≥ 1 | `1` | Page number |
| `per_page` | integer ≥ 1 | all | Results per page |

Returns a paginated `PaginationResults` object with `results`, `total_items`, `current_page`, `total_pages`, `results_on_current_page`, and `results_per_page`.

## Job Status

```
GET /action-creator/submissions/{submission_id}
Authorization: Bearer <token>
```

Optional query parameter: `?workspace=<workspace>`.

Returns an `ActionCreatorJobSummary` with `submission_id`, `status`, `function_identifier`, `submitted_at`, `finished_at`, and `successful`.

### Job status values

| Status | Meaning |
| ------ | ------- |
| `submitted` | Job queued, not yet running |
| `running` | Job is executing |
| `cancel-request` | Cancellation has been requested |
| `successful` | Job completed successfully |
| `failed` | Job failed |
| `cancelled` | Job was cancelled |

## Cancelling or Deleting Jobs

### Single job

```
DELETE /action-creator/submissions/{submission_id}
Authorization: Bearer <token>
```

Returns `204 No Content` on success.

### Batch

```
DELETE /action-creator/submissions
Authorization: Bearer <token>
Content-Type: application/json
```

Request body:

```json
{
  "workspace": "my-workspace",
  "remove_statuses": ["failed", "cancelled"],
  "remove_all_before": "2024-01-01T00:00:00Z",
  "remove_all_after": null,
  "remove_jobs_without_results": false,
  "max_jobs_to_process": 1000
}
```

All fields are optional. `max_jobs_to_process` caps how many jobs are affected in a single call (default 1000).
