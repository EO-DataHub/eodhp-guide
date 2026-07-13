---
title: RC UI Orders Dashboard
doc_status: ok
tags:
  - rc-ui
  - stac
  - commercial-data
  - data-catalogues
last_reviewed:
reviewed_by:
review_notes: Updated from eodhp-rc-ui source. Originally drafted from issues #93, #123.
---

# RC UI Orders Dashboard

Describes the dedicated orders dashboard in the Resource Catalogue UI — its purpose, what it shows, how data is fetched, and the performance improvements made after large-scale order ingestion exposed pagination and loading problems.

---

## What it is

The orders dashboard is at `/my-data/purchased` (within the My Data section alongside Workflow Outputs and Data Storage tabs). It shows all of a user's commercial data orders across all their workspaces with their current status.

---

## Order status values

Status comes directly from the STAC item's `properties['order:status']` field, set server-side. The UI maps it to readable text:

| API value | Displayed as |
|-----------|-------------|
| `pending` | Pending |
| `processing` | Processing |
| `succeeded` | Delivered (data is available) |
| `failed` | Failed |

---

## What the dashboard shows

The table has fixed columns (always visible) and optional columns the user can toggle and reorder. Column preferences are persisted to localStorage.

**Always visible:** Title, Workspace, Collection, View (action button)

**Visible by default:** Order Date, Order Status

**Optional (hidden by default):** Bundle, Country, License, Description, Order ID, Acquisition Date

---

## Filtering and sorting

| Filter | Default | Behaviour |
|--------|---------|-----------|
| Workspace | All Workspaces | Dropdown; persisted to localStorage |
| Collection | All Collections | Dropdown |
| Include Failed Orders | Off | Checkbox; when off, failed orders are hidden |
| Text search | — | Searches across workspace, collection title, item title, order ID, status, bundle, country, license |

Orders are sorted by date descending (newest first). A **Reset Filters** button clears all filters and removes the workspace preference from localStorage.

---

## Viewing data on the map

Each order row has a **View on map** button. Clicking it:

1. Updates the active workspace context to the order's workspace.
2. Stores the item and collection metadata to localStorage (`itemToPin`).
3. Navigates to `/finder/my-{collectionId}`.

The Data Finder page picks up `itemToPin` on mount, loads the item, and pins it on the map with the pinned items tab open. A separate **View Collection** button navigates to the same route without pinning a specific item.

---

## How data is fetched

Fetching is hierarchical across all workspaces:

1. `GET /api/workspaces` — list all the user's workspaces.
2. `POST /api/workspaces/{workspace}/me/sessions` — get a workspace-scoped token for each workspace.
3. `GET /api/catalogue/stac/catalogs/user/catalogs/{workspace}/catalogs/commercial-data/collections` — list purchased collections in that workspace.
4. `GET .../collections/{collectionId}/items?limit=200` — fetch items (orders) from each collection, following `rel="next"` cursor links for additional pages.

To avoid overwhelming the browser, fetching is concurrency-limited: a maximum of 2 workspaces and 3 collections per workspace are fetched simultaneously.

---

## Pagination

There are two independent levels of pagination:

**API-level (per collection):** Up to 200 items are fetched per collection per request. If a collection has more, a **Load More** button appears. Subsequent pages are fetched via the STAC API's cursor-based `next` link.

**Client-side (display):** The filtered and sorted order list is paginated at 10 items per page. A page selector shows "Showing X–Y of Z orders" (with a `+` suffix when the total is unknown because more pages haven't been loaded yet, and a "(filtered)" suffix when any filters are active).
