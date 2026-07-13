---
title: Processes Catalogue
doc_status: ok
tags:
  - rc-ui
  - stac
  - workflows
last_reviewed:
reviewed_by:
review_notes: Updated from workflow-catalogue and processes-catalog-api-proposal repos. Issue #41 still open.
---

# Processes Catalogue

Describes the design intent, API standard, and integration points for the EODH Processes Catalogue — a discoverable registry of reusable workflows and notebooks.

See also: [Parameterised Notebook and Workflow Launch](parameterised-notebook-and-workflow-launch.md) — how the RC UI launches notebooks and workflows from a selected STAC item.

---

## What it is

The Processes Catalogue is a library of reusable EO workflows and Jupyter notebooks that users can discover and launch directly from the Resource Catalogue UI. It complements the STAC-based data catalogue by providing a parallel catalogue for *processing operations* rather than datasets.

Goals:
- Make workflows and notebooks discoverable alongside the data they operate on.
- Allow users to browse and select a workflow or notebook, then launch it against a chosen STAC collection.
- Support partner/external workflow providers publishing their algorithms into the catalogue.

---

## Standard: OGC API Records

The catalogue aligns with the [OGC API - Records](https://ogcapi.ogc.org/records/) standard, with EODH-specific extensions. All records declare:

```json
"conformsTo": ["http://www.opengis.net/doc/IS/ogcapi-records-1/1.0"]
```

Key OGC Records concepts used:

| Concept | Role in processes catalogue |
|---------|-----------------------------|
| **Record** | Represents a single workflow or notebook |
| **Collection** | Groups of related records (e.g. `eodh-workflows-notebooks`) |
| **Links** | References to the execution endpoint, CWL/notebook file, source repo |
| **Properties** | Metadata: title, description, applicable collections, version, contacts, licence |

---

## Architecture: Git-based catalogue with API backend

The catalogue is managed as a **Git repository** (`workflow-catalogue`) rather than a live database. JSON record files are committed to the repo and published automatically via CI/CD to a backend API service (`wf-catalogue-service`). This gives a clean audit trail through Git history and allows schema validation before anything reaches the API.

```
workflow-catalogue (Git repo)
  catalogue/
    {collection_id}/
      catalog.json          ← collection metadata
      workflows/
        {workflow_id}.json  ← workflow records
      notebooks/
        {notebook_id}.json  ← notebook records
```

On merge to `main`, the CD pipeline:
1. Creates or updates the collection in `wf-catalogue-service` if needed.
2. Registers each changed record via the service API.
3. For workflows: registers the CWL process in ADES and triggers a harvest to publish it.

Merge to `develop` targets the dev environment; `main` targets production.

---

## Record types

The catalogue supports two record types: **workflows** and **notebooks**. Both follow the same OGC Records base structure with type-specific extensions.

### Workflow records

Workflows are CWL-based processes executed via ADES.

**Required properties:**

| Field | Description |
|-------|-------------|
| `id` | Unique identifier (alphanumeric, hyphens, underscores) |
| `type` | `"Feature"` |
| `properties.type` | `"workflow"` |
| `properties.title` | Human-readable name |
| `properties.description` | What the workflow does |
| `properties.keywords` | One or more searchable tags |
| `properties.language` | ISO 639-1 language code (e.g. `"en"`) |
| `properties.license` | SPDX licence identifier (e.g. `"Apache-2.0"`) |
| `properties.applicableCollections` | Array of STAC collection IDs this workflow can operate on |
| `properties.contacts` | One or more contact objects (name, organisation, roles) |
| `properties.inputParameters` | Map of parameter name → parameter definition (see below) |
| `properties.application:type` | Fixed: `"cwl"` |
| `properties.application:container` | `true` |
| `properties.application:language` | Fixed: `"CWL"` |
| `properties.created` / `updated` | ISO 8601 timestamps |
| `links` | Must include `application-platform` (ADES URL), `application` (CWL file), `vcs` (source repo) |

**Input parameter types** (`properties.inputParameters`):

Each parameter entry requires `label`, `type`, and `description`. Supported types: `raster`, `vector`, `collection`, `catalog`, `netcdf`, `geotiff`, `wms`, `wfs`, `bbox`, `date`, `text`, `number`, `boolean`.

Optional per-parameter fields: `default`, `placeholder`, `required`, `min`, `max`, `pattern`, `enum`.

### Notebook records

Notebooks are Jupyter notebooks executed in JupyterHub.

Same base structure as workflows with these differences:

| Field | Value |
|-------|-------|
| `properties.type` | `"notebook"` |
| `properties.application:type` | `"jupyter-notebook"` |
| `properties.application:container` | `false` |
| `properties.application:language` | Programming language string (e.g. `"Python"`) |
| `properties.license` | Optional (not required) |
| `properties.jupyter_kernel_info` | Object with kernel name, Python version, env file URL |
| `properties.formats` | Array of output format objects with `name` and `mediaType` |

---

## API endpoints (wf-catalogue-service)

The `wf-catalogue-service` backend exposes these endpoints:

```
GET  /api/v1.0/collections                         # List all collections
POST /api/v1.0/collections                         # Create a collection
GET  /api/v1.0/collections/{collection_id}         # Get collection metadata
DELETE /api/v1.0/collections/{collection_id}       # Delete a collection
POST /api/v1.0/register?catalogue_id={id}          # Register a record
DELETE /api/v1.0/register/{record_id}              # Delete a record
GET  /api/v1.0/collections/{collection_id}/items   # List records (with filtering)
GET  /health                                        # Health check (unauthenticated)
```

Filtering on `/items`:
- `?type=workflow|notebook` — filter by record type
- `?q=search_term` — full-text search
- `?keywords=keyword` — filter by keyword
- `?page=` / `?page_size=` — pagination

The following endpoints are publicly accessible (no auth required): `/health`, `GET /api/v1.0/collections`, and `GET /api/v1.0/collections/{collection_id}/items`. All write and delete endpoints require a Bearer token (`Authorization: Bearer {token}`) via OAuth2/Keycloak.

---

## How the RC UI uses the processes catalogue

The UI queries a single fixed collection: `eodh-workflows-notebooks`. All workflow and notebook records must be registered under that collection ID to be visible in the UI.

### Catalogue browse page

A dedicated browse page at `/catalogue/processes` lists all records with search and filtering:

- **Text search** — full-text via `?q=` (400ms debounce)
- **Type filter** — All / Workflow / Notebook
- **Collection filter** — dropdown of all STAC collections; filters by `applicableCollections`

The detail page for a record (at `/catalogue/processes/{recordId}`) shows the full record including title, description, keywords, applicable collections, contacts, and links. It does not have a launch button — instead it directs the user to the Data Finder to run it.

### Contextual integration (Data Finder)

When a user selects a dataset item in the Data Finder, an "Integration Tools" panel opens with **Workflows** and **Jupyter** tabs. The UI automatically filters records using `applicableCollections` to match the collection of the selected item — only relevant workflows and notebooks are shown.

### Launching a workflow

Selecting a workflow opens a launch modal with:

1. **Collection selector** — pre-populated from the workflow's `applicableCollections`, filtered to collections that exist in the catalogue.
2. **Dataset selector** — STAC items from the selected collection, with optional date range filtering.
3. **AOI checkbox** — if an area of interest is drawn on the map.
4. **Input parameter form** — fields generated from `inputParameters`.

On submit, the UI calls the ADES execution endpoint declared in the `application-platform` link:

```
POST {HUB_BASE_URL}/api/catalogue/stac/catalogs/user/catalogs/{workspace}/processes/{processId}/execution
```

### Launching a notebook

Notebook launch extracts a template URL from the record's links (`rel="template"`, falling back to `rel="application"` then `rel="enclosure"`), spawns a JupyterLab instance, and opens it in a new tab.

### Input parameter variable references

Some input parameters are auto-resolved from the user's current selection rather than shown as form fields. Set a `variableRef` on the parameter to enable this:

| `variableRef` value | Resolved to |
|---------------------|-------------|
| `STAC_ITEM_LINK` | The `self` link of the selected STAC item |
| `STAC_COLLECTION_NAME` | The collection ID of the selected item |
| `AOI` | GeoJSON Feature of the user's drawn area of interest |

Parameters with a `variableRef` are passed silently at launch; parameters without one are shown as form inputs to the user.

### Fields rendered by the UI

| Context | Fields used |
|---------|-------------|
| Browse card | `properties.type`, `properties.title`, `properties.description` (truncated to 120 chars) |
| Detail page | All of the above + `properties.keywords`, `properties.applicableCollections`, `properties.contacts`, `links` |
| Launch modal | `properties.title`, `properties.description`, `properties.applicableCollections`, `properties.inputParameters`, `links[rel=application-platform]` |

---

## Registering a new workflow or notebook

The submission process is Git-based:

1. Fork or branch the `workflow-catalogue` repository.
2. Add a JSON record file under `catalogue/{collection_id}/workflows/` or `.../notebooks/`. The filename must match the `id` field (e.g. `ndvi-workflow.json` must have `"id": "ndvi-workflow"`).
3. Run local validation: `python -m workflow_catalogue validate <file>`.
4. Open a pull request. CI validates the JSON schema (Pydantic), checks that `applicableCollections` URLs are reachable, and validates CWL syntax via `cwltool`.
5. On merge to `main`, the CD pipeline registers the record with `wf-catalogue-service` and (for workflows) deploys the CWL to ADES.

A workflow becomes discoverable in the catalogue once CD completes successfully.
