---
title: Parameterised Notebook and Workflow Launch
doc_status: ok
tags:
  - jupyter
  - notebooks
  - workflows
  - stac
last_reviewed:
reviewed_by:
review_notes: Drafted from issues #38, #136, #139, #140, #141, #142, #143, #145. Updated from eodhp-rc-ui source.
---

# Parameterised Notebook and Workflow Launch

Explains how the notebook and workflow launch features work — allowing users to open a pre-populated Jupyter notebook or trigger an ADES workflow directly from a STAC item in the Resource Catalogue.

See also: [Processes Catalogue](processes-catalogue.md) — how notebook and workflow records are registered and structured.

---

## Where launch is triggered from

Both notebook and workflow launch are accessed from the **Integration Tools** panel in the Data Finder. When a user selects a STAC item, a side panel opens with three tabs:

- **QGIS** — download a QGIS layer file or copy an XYZ tile URL for the item
- **Workflows** — workflows applicable to the item's collection
- **Jupyter** — notebooks applicable to the item's collection

The applicable workflows and notebooks are fetched from the processes catalogue filtered by the item's collection ID (`applicableCollections`). Only records that declare the relevant collection will appear.

The processes catalogue browse page (`/catalogue/processes`) is informational — the detail page for a record shows metadata but directs users to the Data Finder to actually run anything.

---

## Notebook launch

### User flow

1. User selects a STAC item in the Data Finder.
2. User opens **Integration Tools → Jupyter** tab and selects a notebook.
3. Optionally ticks "Include area of interest" if an AOI is drawn on the map.
4. User clicks **Launch**.
5. The frontend resolves input variables from the selected item and calls the parameterisation API.
6. The API fetches the notebook template, injects parameter values, uploads the result to the user's workspace S3, and returns a JupyterHub spawn URL.
7. The spawn URL opens in a new browser tab, landing the user directly in the populated notebook.

### Components

| Component | Repository | Role |
|-----------|-----------|------|
| Integration Tools panel | `eodhp-rc-ui` | Displays applicable notebooks, handles launch |
| Parameterisation API | `resource-catalog-support-utils` | Injects parameters, uploads to S3, returns spawn URL |
| Notebook templates | S3 / accessible URL | Reusable `.ipynb` files with parameter placeholders |
| Notebook records | `workflow-catalogue` | Maps notebooks to templates and applicable collections |

### API endpoint

**`POST {RC_SUPPORT_UTILS_URL}/api/v1/notebooks/launch`**

**Request:**
```json
{
  "workspace_name": "sparkgeotest",
  "username": "geodowd",
  "action_name": "ndvi-notebook",
  "template_url": "https://example.com/templates/ndvi_template.ipynb",
  "variables": {
    "STAC_ITEM_LINK": "https://eodatahub.org.uk/api/catalogue/stac/.../items/...",
    "STAC_COLLECTION_NAME": "sentinel2_ard"
  }
}
```

`action_name` is the notebook record's `id` from the processes catalogue. `template_url` is extracted from the record's links (priority: `rel="template"` → `rel="application"` → `rel="enclosure"`). `variables` is populated by resolving the record's `inputParameters` against the selected item (see [Variable resolution](#variable-resolution) below).

**Response:**
```json
{
  "servername": "ndvi-notebook_123456",
  "notebook_path": "jupyter/ndvi-notebook_123456.ipynb",
  "spawn_url": "https://eodatahub-workspaces.org.uk/notebooks/hub/spawn/geodowd/ndvi-notebook_123456?workspace=sparkgeotest&next=..."
}
```

### S3 upload location

Parameterised notebooks are uploaded to:

```
s3://workspaces-eodhp/<workspace_name>/jupyter/<servername>.ipynb
```

The server name follows the pattern `<action_name>_<unique_id>` (e.g. `ndvi-notebook_123456`).

### JupyterHub spawn URL format

```
https://eodatahub-workspaces.org.uk/notebooks/hub/spawn/<username>/<servername>
  ?workspace=<workspacename>
  &next=https://eodatahub-workspaces.org.uk/notebooks/user/<username>/<servername>/lab/tree/s3/<path_to_notebook>
```

Using a named server (`servername`) avoids conflicts with any existing running JupyterHub server the user has open. Each launch creates a fresh named server instance.

**Note on server state:** If the spawn URL is used while the named server is already stopped, JupyterHub will prompt the user to start it before redirecting to the notebook. This is expected behaviour.

---

## Workflow launch

### User flow

1. User selects a STAC item in the Data Finder.
2. User opens **Integration Tools → Workflows** tab and selects a workflow.
3. A launch modal opens showing the workflow title and description.
4. The modal presents:
   - **Collection selector** — pre-populated from `applicableCollections`, filtered to collections that exist in the catalogue.
   - **Date range inputs** (optional) — to filter the dataset selector.
   - **Dataset selector** — STAC items from the selected collection.
   - **AOI checkbox** — if an area of interest is drawn on the map.
   - **Input parameter form** — fields for any `inputParameters` without a `variableRef`.
5. User clicks **Launch**.
6. The frontend acquires a workspace-scoped token, then submits the job to ADES.
7. On success, the user sees the job ID with a link to the workflow outputs page.

### ADES execution request

The frontend acquires a workspace-scoped token before submission:

```
POST {HUB_BASE_URL}/api/workspaces/{workspace}/me/sessions
```

It then submits to the execution endpoint declared in the workflow record's `application-platform` link:

```
POST {HUB_BASE_URL}/api/catalogue/stac/catalogs/user/catalogs/{WORKFLOW_RESULTS_WORKSPACE}/processes/{processId}/execution
```

**Request body:**
```json
{
  "inputs": {
    "STAC_ITEM_LINK": "https://eodatahub.org.uk/api/catalogue/stac/.../items/...",
    "STAC_COLLECTION_NAME": "sentinel2_ard",
    "my_param": "user-supplied value"
  }
}
```

Auto-resolved variables are merged with user-supplied values; user-supplied values take precedence.

The execution call returns **202 Accepted** — workflows are asynchronous. After submission the user is directed to:

```
/my-data/workflow-outputs/{workspace}/{workflowId}
```

---

## Variable resolution

Both notebook and workflow launch resolve `inputParameters` from the selected item before calling the API. Parameters with a `variableRef` are resolved automatically and not shown as form fields:

| `variableRef` | Resolved to |
|---------------|-------------|
| `STAC_ITEM_LINK` | The `self` link href of the selected STAC item |
| `STAC_COLLECTION_NAME` | The `collection` ID of the selected item |
| `AOI` | JSON-stringified GeoJSON Feature of the user's drawn area of interest |

Parameters without a `variableRef` are presented as form inputs in the launch modal.

---

## Notebook templates

Templates are standard `.ipynb` files. The parameterisation API replaces `{{PLACEHOLDER}}` values in cell source before uploading:

| Placeholder | Replaced with |
|-------------|--------------|
| `{{STAC_COLLECTION_NAME}}` | The STAC collection ID |
| `{{STAC_ITEM_LINK}}` | The STAC item self-link URL |

Example cell:

```python
STAC_COLLECTION = "{{STAC_COLLECTION_NAME}}"
STAC_ITEM_URL   = "{{STAC_ITEM_LINK}}"
```

### Creating a new template

1. Create a `.ipynb` notebook that performs the desired analysis.
2. Replace hardcoded dataset references with `{{STAC_ITEM_LINK}}` or `{{STAC_COLLECTION_NAME}}` placeholders.
3. Include a markdown cell at the top explaining what the notebook does and what the injected parameters represent.
4. Upload or commit the template to an accessible URL.
5. Register it in the processes catalogue — add a notebook record to `workflow-catalogue` with a `rel="template"` link pointing to the template URL. See [Processes Catalogue — Registering a new workflow or notebook](processes-catalogue.md#registering-a-new-workflow-or-notebook).

---

## Known limitations

### Private STAC records require manual token setup (#143)

Users who want to access **private STAC records** (purchased data, processed results) from within a launched notebook must manually create an `.env` file in their workspace containing their authentication token. This is not done automatically by the parameterised launch flow.

Options being considered:
- Inject the token as a notebook parameter at launch time.
- Use the `python-keycloak` SDK (or a custom function) to fetch a token at notebook runtime via API keys.
- Expose the token as a pre-configured environment variable on the JupyterHub single-user server.

Until this is resolved, notebooks requiring access to private assets should include a prominent cell directing the user to set up their credentials. See the [workshop notebook pattern](https://github.com/EO-DataHub/eodh-training/blob/main/presentations/Workshop/2_202502_workshop_DataProcessing.ipynb) for the current workaround.

### Server lifecycle

Named JupyterHub servers are culled after idle timeout. A spawn URL embedded in a bookmark or shared link may redirect the user to the "start server?" page rather than directly into the notebook — this is expected and not an error.
