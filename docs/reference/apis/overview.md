---
title: 01. Overview
doc_status: ok
last_reviewed: 2026-06-01
reviewed_by: geodowd
review_notes: "Updated from live production OpenAPI spec at eodatahub.org.uk/api/docs/openapi.json"
tags:
  - api
---
# 01. Overview

EODH provides an API using REST principles rooted at https://eodatahub.org.uk/api/ which is used to expose access to metadata and data services through STAC and OGC APIs, plus access to EODH platform functionality such as workspace management through custom APIs. In particular, the API should follow the following principles:

* Public APIs are a user interface and must make sense from a user perspective
* Users should not see the joins between components inside EODH nor other implementation details
* The APIs should follow REST principles and those behind the new OGC APIs, including:
	* URLs represent resources, not actions or queries
	* Proper use of HTTP verbs, status codes, content negotiation, etc.
	* APIs can be navigated via links

## API Structure

Under https://eodatahub.org.uk/:

|                                       |                                                                                        |
| ------------------------------------- | -------------------------------------------------------------------------------------- |
| **/api/**                             | Landing page                                                                           |
| **/api/catalogue/stac/**              | Metadata and data access APIs (STAC v1 / OGC Records / OGC Features)                  |
| **/api/workspaces/**                  | Workspace management                                                                   |
| **/api/accounts/**                    | Account management                                                                     |
| **/api/accounting/**                  | Billing SKU catalogue and current prices                                               |
| **/api/wf-catalogue/**                | Workflow Catalogue — browse, register and manage workflow/notebook records              |
| **/titiler/**                         | Visualization and mapping — COG/GeoTiff, Zarr/xarray, tile matrix sets                |
| **/.well-known/openid-configuration** | OIDC information - endpoints under **/keycloak/realms/eodhp/protocol/openid-connect/** |

Under https://{workspace-name}.eodatahub-workspaces.org.uk/:

|                                         |                                                                        |
| --------------------------------------- | ---------------------------------------------------------------------- |
| **/**                                   | File access to workspace primary object store                          |
| **/files/{store-name}/**                | File access to other workspace stores                                  |
| (possible future) **/api/catalogue/stac/** | Metadata and data access APIs containing only this workspace's entries |

These endpoints are described individually below.

## Authentication and Authorization

Workspaces are similar to a tenancy in many systems and for many calls it's important that a unique workspace is identified, including anything which may generate billing events. For that reason, the authentication and authorization used by an API endpoint may be of two types:
* **User-scoped**: No specific workspace is identified and only API endpoints not requiring one can be called. This includes full access to **/api/workspaces** and **/api/accounts**, plus read access to STAC and Records endpoints in **/api/catalogue** (access to OGC Services TBD - this depends on how associated resource use will be billed-for). The web presence's cookie is user-scoped. User-scoped API Tokens and OAuth2 Tokens are conceptually possible and could be useful for some future applications but are not initially issued.
* **Workspace-scoped**: A specific (single) workspace is identified and only API calls accessing resources 'in' that workspace are permitted. This allows full access to **/api/catalogue/** and read-only access to **/api/workspaces/** with only the single workspace visible. API Tokens and OAuth2 Tokens are normally workspace-scoped. Cookies are never workspace-scoped.

User-scoped access is intended for managing workspaces and accounts whilst workspace-scoped access is intended for the core functionality of the system.

Access to metadata and data services will generally return a 404 where it identifies a specific private resource a user cannot access or return a filtered or empty list. 403 may be returned in more limited circumstances such as when a resource cannot be accessed due to limited access token scope.

## Endpoints

### Root - **/api/**

|                   |                                                                                                                                                                                                                                                                                                                                                        |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **/api/**         | When JSON is requested, and OGC Features API landing page with `links: [ ... ]` pointing to each service, eg one with `rel="http://www.opengis.net/def/rel/ogc/1.0/ogc-catalog"` pointing to **/api/catalogue/stac** one with `rel="https://api.stacspec.org/v1.0.0/core"` pointing there, too). When HTML is requested an introduction page to the API. |
| **/api/api**      | OpenAPI document describing the APIs -  links to **/api/catalogue/stac/api** rather than including it.                                                                                                                                                                                                                                                   |
| **/api/api.html** | OpenAPI reader pointed at **/api/api**                                                                                                                                                                                                                                                                                                                 |
 
### Catalogue - **/api/catalogue/stac/**

#### Correspondence Between STAC, OGC Records and OGC Features API

Whilst both STAC and OGC Records API are explicitly built on top of the OGC Features API, there is no explicit relationship between STAC and OGC Records themselves defined in the standards. However, the APIs can be combined such that some endpoints are both OGC Records API-compliant and STAC API-compliant, and some endpoints are specifically STAC APIs or STAC extensions. The primary goal is to ensure that unmodified STAC clients can use the API fully without encountering problems. Secondary goals are to support OGC Records API clients and to support holding non-dataset entries following the Records standard sufficiently that API clients designed for use with EODH can access them.

The core entities in the three standards have a correspondence:

| STAC       | OGC Records            | OGC Features       |
| ---------- | ---------------------- | ------------------ |
| Catalog    | Landing page           | Landing page       |
| Collection | Record Collection      | Feature collection |
| Item       | Record of type Feature | Feature            |
| Item(\*)   | Record of other types  | Feature            |

This is slightly different to the description at https://github.com/stac-utils/stac-crosswalks/blob/master/ogcapi-records/README.md which associates STAC Catalogs with OGC Collections.

Item(\*) refers to a record which is primarily considered an OGC Records Record but with some additional fields to make it sufficiently STAC compliant to avoid breaking STAC clients which are using the API. For example, this includes setting `geometry` to `null` and adding minimal `stac_version`, `stac_extensions` and `assets` fields.

##### STAC and OGC Records Correspondence Details - Catalog and Landing Page

A page such as **/api/catalogue/stac/catalogs/public/** is an EODH Catalog landing page, meaning that it's a JSON file with fields:
* `title`, `description` and `links` from OGC Features API and OGC Records API
* `id`, `type`, `stac_version` and `conformsTo` from STAC and STAC API, plus `rel=child` links in `links` to child Catalogs.

**/conformance** likewise conforms to all three standards.

##### STAC and OGC Records Correspondence Details - Collections Search

A page such as **/api/catalogue/stac/catalogs/public/collections** conforms to all three standards. This is a Features API Feature Collections endpoint that can be used for searching and listing collections. The Collections listed in the `collections: []` field may each either be only an OGC Records API Record, or both an OGC Records API Record Collection and a STAC Collection.

OGC Records are only used for non-spatiotemporal entries such as workflows or applications and have a corresponding `type` field, eg `process`.

STAC Collections/OGC Record Collections have `"type": "Collection"` to conform with STAC and  `itemType` set to `"record"`, `"catalog"` or `["record", "catalog"]` depending on whether the collection contains only items, only sub-collections or both. In EODH we expect this to only ever be `"record"` because nested Collections are not supported by stac-fastapi.

##### STAC and OGC Records Correspondence Details - Collections

STAC Collections/OGC Record Collections, in both search results and locations such as **/api/catalogue/stac/catalogs/public/catalogs/ceda/collections/cmip6**, have fields defined in both the OGC Records API and in STAC. For example (this is not exhaustive):

* `type`, `stac_version`, `stac_extensions`, `summaries`, `item_assets` and `assets` come from STAC alone
* `properties.type`, `itemType`, `crs`, `storageCRS`, `linkTemplates` and `themes` are defined in OGC Records only
* `id`, `title`, `description`, `links`, `keywords`, `extent`, `license`, `created`, `updated` are defined in both STAC and OGC Records

Potential clashes:
* OGC Records API defines (https://docs.ogc.org/DRAFTS/20-004.html#collection-properties-table) `type` for a Record Collection to be `Catalog` (OGC Records uses 'collection of records' and 'catalog' interchangeably). STAC defines it to be `Collection`. EODH uses `Collection`.

##### STAC and OGC Records Correspondence Details - Items

The items search endpoints, such as **/api/catalogue/stac/catalogs/public/catalogs/ceda/collections/cmip6/items**, are extended with STAC API extensions such as the filter extension, which adds support for CQL. It's also a valid Records API records access endpoint supporting `limit`, `bbox`, etc. (which are also know to STAC) and a Features API 'features' endpoint.

As with collections, fields are defined in one or both standards (and some from GeoJSON). For example (not exhaustive):

* `type`, `stac_version`, `stac_extensions` and `assets` come from STAC alone (except that `type` is from GeoJSON and fixed to `Feature`).
* `properties.type`, `properties.rights`, `properties.externalIds` and `time` are defined in OGC Records only
* `id`, `properties.title`, `properties.description`, `links`, `properties.license`, `properties.created`, `properties.updated` are defined in both STAC and OGC Records

##### STAC and OGC Records Correspondence Details - non-STAC Records

A result from a collection search or collection endpoint (`.../collections`) may have a type other than `Collection` (or `Catalog`) and so not be a STAC Collection.

#### EODH Catalog Endpoints

An EODH Catalog endpoint, through its root and various sub-paths, provides access to metadata and data using a variety of OGC standards, the STAC standard and some EODH extensions. EODH Catalogs are nested - Catalogs have sub-Catalogs, nesting to an arbitrary depth (though in practice rarely more than 5 levels). Any EODH Catalog at any level in the hierarchy behaves the same way, so any well-written client can use any such endpoint without needing to be aware of its exact position in the catalogue.

The location of an entity within the catalogue is referred to using a 'catalogue path' or cat-path. For example, `/catalogs/public/catalogs/ceda/collections/cmip6/items/neodc.sentinel_ard.data.sentinel_2.2023.11.21.S2B_20231121_latn536lonw0052_T30UUE_ORB123_20231121122846_utm30n_TM65`  is the cat-path of an item. The non-fixed elements of the path always match the corresponding id of the entity.

The catalogue is arranged into a hierarchy of Catalogs as follows, although user Catalogs can be extended arbitrarily:

| EODH Catalog Endpoint                                                                 |                                               |
| ------------------------------------------------------------------------------------- | --------------------------------------------- |
| **/api/catalogue/stac/**                                                                | EODH root                                     |
| **/api/catalogue/stac/catalogs/public/**                                                | EODH public data                              |
| **/api/catalogue/stac/catalogs/commercial/**                                            | EODH commercial data                          |
| **/api/catalogue/stac/catalogs/workflows/**                                             | EODH-supplied public workflows                |
| **/api/catalogue/stac/catalogs/user/**                                                  | User data - visible sub-Catalogs are filtered |
| **/api/catalogue/stac/catalogs/user/catalogs/my-workspace/**                            | User data for my-workspace                    |
| **/api/catalogue/stac/catalogs/user/catalogs/my-workspace/catalogs/saved-data**         | Data saved                                    |
| **/api/catalogue/stac/catalogs/user/catalogs/my-workspace/catalogs/commercial-data**    | Previously ordered commercial data            |
| **/api/catalogue/stac/catalogs/user/catalogs/my-workspace/catalogs/processing-results** | Default location for workflow outputs         |

The data and metadata accessible to you will depend on the access policies set for each Catalog and for each Collection contained in them. The contents of `public` and `commercial` Catalogs will usually be completely open for metadata access without authentication. `user` Catalogs are owned by a particular user workspace and are normally private to its members, but workspace members can choose to apply public access to parts of their workspace Catalog.

Within a Catalog endpoint a variety of other endpoints are available, summarized here:

| Path                                                                                                                                         | Standards                   |                                                                                                        |
| -------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------- | ------------------------------------------------------------------------------------------------------ |
| **.../api**                                                                                                                                  | OpenAPI                     | JSON API description                                                                                   |
| **.../api.html**                                                                                                                             | -                           | API documentation page                                                                                 |
| **.../conformance**                                                                                                                          | Features, Records, STAC API | Conformance list                                                                                       |
| **.../search**                                                                                                                               | STAC API extension          | Cross-collection Item search                                                                           |
| **.../catalogs**                                                                                                                             | -                           | Catalog search                                                                                         |
| **.../collections**                                                                                                                          | Features, Records, STAC     | Collection search                                                                                      |
| **.../collections/{collection-id}**                                                                                                          | Features, Records, STAC     | Collection by ID                                                                                       |
| **.../collections/{collection-id}/items**                                                                                                    | Features, Records, STAC     | Single-collection Item search                                                                          |
| **.../collections/{collection-id}/items/{item-id}**                                                                                          | Features, Records, STAC     | Single Item                                                                                            |
| **.../collections/{collection-id}/items/{item-id}/quote**                                                                                    | -                           | Returns a price quote or estimate - for commercial data only (could later extent to other entry types) |
| **.../queryables**, **.../collections/{collection-id}/queryables**                                                                           | STAC filter extension       | Queryables that search can filter by                                                                   |
| **.../\[aggregates\|aggregations]**, **.../collections/{collection-id}/\[aggregates\|aggregations]**                                         | STAC aggregation extension  | Aggregate information about STAC Items                                                                 |
| **.../collections/{collection-id}/annotations**                                                                                              | -                           | EODH annotations search                                                                                |
| **.../collections/{collection-id}/annotations/{annotations-graph-id}**                                                                       | -                           | Single EODH annotations graph                                                                          |
| **.../access-policy**, **.../collections/{collection-id}/access-policy**                                                                     | -                           | (internal) Catalogue access control configuration                                                      |
| **.../wmts**                                                                                                                                 | OGC WMTS                    | Map tiles for Collections with STAC Render extension in the Catalog (via TiTiler)                      |
| **.../layers/{LAYER}/{STYLE}/{TIME}/{TileMatrixSet}/{TileMatrix}/{TileCol}/{TileRow}.{FORMAT}**                                              | OGC WMTS GetTile (REST)     | Map tiles for Collections with STAC Render extension in the Catalog (via TiTiler)                      |
| **.../collections/{collection-id}/tiles**, **.../collections/{collection-id}/items/{item-id}/tiles**                                         | OGC Tiles API               | Map tiles for Collections in the Catalog (via TiTiler)                                                 |
| **.../collections/{collection-id}/{tile-matrix-set-id}/map**                                                                                 |                             | Basic map viewer (via TiTiler)                                                                         |
| **.../collections/{collection-id}/{tile-matrix-set-id}/tilejson.json**                                                                       | TileJSON                    | Mapbox TileJSON file                                                                                   |
| **.../collections/{collection-id}/{tile-matrix-set-id}/WMTSCapabilities.xml**                                                                | OGC WMTS                    | GetCapabilities result for the Collection                                                              |
| **.../collections/{collection-id}/items/{item-id}/{bounds,info,info.geojson,assets,asset_statistics,statistics,point,preview,bbox,feature}** |                             | Item information endpoints (TiTiler)                                                                   |
| **.../tileMatrixSets**, **.../tileMatrixSets/tile-matrix-set-id}**                                                                           |                             | Available tile matrix sets for use with APIs above                                                     |
| **.../algorithms**, **/algorithms/{algorithm-id}**                                                                                           |                             | Algorithms for use with the APIs above                                                                 |
| **.../processes**, **.../processes/{workflow-id}**                                                                                            | OGC Processes API           | List, deploy, update or undeploy workflows in a workspace                                              |
| **.../processes/{workflow-id}/execution**                                                                                                    | OGC Processes API           | POST to execute a workflow; returns a job                                                              |
| **.../jobs**, **.../jobs/{job-id}**                                                                                                          | OGC Processes API           | List all jobs or get/delete a single job                                                               |
| **.../jobs/{job-id}/results**                                                                                                                | OGC Processes API           | GET results of a completed job (returns a STAC Collection)                                             |
| **.../jobs/{job-id}/{file-name}**                                                                                                            | -                           | GET a job log file                                                                                     |

#### Recursive vs Non-Recursive Queries

There is a hole in the STAC API spec concerning whether `<catalog x>/collections` and `<catalog x>/search` search only collections directly within the Catalog or also search collections which are inside sub-Catalogs. Both options have problems.

The EODH approach is as follows:
* `<catalog>/collections`, `<catalog>/search` and `<catalog>/catalogs` are all recursive, meaning they return results from the Catalog that was queried or any sub-Catalog.
* Catalogs contain links pointing to direct-child Collections and Catalogs. API users can distinguish between them without fetching them only by checking if the penultimate path component is `catalogs` or `collections`.
* `<catalog>/catalogs/<catalog id>` and `<catalog>/collections/<collection id>` are non-recursive. This is necessary because collection IDs are not unique across Catalogs.

#### EODH Workflows in the API

Workflows can be added to any catalog's processes endpoint using POST or workflow harvesting, but it's likely users will primary add them to `/api/catalogue/stac/catalogs/user/catalogs/<username>/processes`.

Workflow catalogue entries are separate entities and are optional. If added, they allow the workflow to be found using the catalogue UI. For a workflow such as `/api/catalogue/stac/catalogs/workflows/processes/an-eodh-workflow` the catalogue entry would be found at `/api/catalogue/stac/catalogs/workflows/collections/a-workflow-collection/items/an-eodh-workflow`, with the publisher of the workflow able to decide how to group workflows into multiple collections if desired.

#### Commercial data in the API

Commercial data will be represented in the catalogue as STAC within a special commercial data sub-Catalog, `/api/catalogue/stac/catalogs/commercial`. For example, `/api/catalogue/stac/catalogs/commercial/catalogs/planet/collections/PSScene` represents Planet's PlanetScope data. The STAC Items are searchable using STAC APIs but will not contain the full set of assets. For Planet data, or any other provider where harvesting a complete set of Items is not feasible, the Items will be searchable only through the collections' own item search endpoint and not through the catalogue-wide item search endpoint. Typically with commercial data there is no one-to-one mapping between these Items and the data that users obtain if they place an order - a new Item is generated by the order and will look different to the one in the main catalogue (eg, smaller area).

The live API exposes quote and order as POST endpoints directly on the item, using parameterised paths where `{parent_catalog}` is `commercial` or `supported-datasets` and `{catalog}` is `planet` or `airbus`:

* **Quote**: `POST /api/catalogue/stac/catalogs/{parent_catalog}/catalogs/{catalog}/collections/{collection}/items/{item}/quote` — returns a price and currency.
* **Order**: `POST /api/catalogue/stac/catalogs/{parent_catalog}/catalogs/{catalog}/collections/{collection}/items/{item}/order` — creates an order; accepts `productBundle`, optional `coordinates`, `licence`, and `radarOptions`.

Once accepted, an order will result in a STAC Item being added to a users' ordered data catalogue in their workspace catalogue.

This STAC Item will use the STAC Order extension to describe order status and will be updated upon success or failure of the order.

### EODH Workspaces and Accounts API Endpoints

These are typical REST-style interfaces. It's expected that most endpoints will not be used directly by users or applications except for obtaining tokens and workspace information. The others are used by the UI.

|                                                                      |                                                                                                                                                                   |
| -------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **/api/accounts/**                                                   | GET lists billing accounts owned by the authenticated user                                                                                                        |
| **/api/accounts/{account-id}**                                       | GET account info (includes list of workspaces)                                                                                                                    |
| **/api/accounts/{account-id}/accounting/usage-data**                 | GET resource consumption data for all workspaces in an account (filterable by time range, paginated, supports `day`/`month` aggregation)                          |
| ...                                                                  |                                                                                                                                                                   |
| **/api/workspaces/**                                                 | GET lists accessible workspaces, POST to create                                                                                                                   |
| **/api/workspaces/{workspace-id}**                                   | GET for workspace info, PUT to create or update, PATCH to update. Workspace info includes some information about the owning account.                              |
| **/api/workspaces/{workspace-id}/files**                             | GET to list files across object and/or block stores (`?store=object\|block`)                                                                                      |
| **/api/workspaces/{workspace-id}/files/block**                       | POST to upload files to the block store, DELETE to remove                                                                                                         |
| **/api/workspaces/{workspace-id}/files/block/metadata**              | GET metadata for a single block store file                                                                                                                        |
| **/api/workspaces/{workspace-id}/files/object**                      | POST to upload files to the object store, DELETE to remove                                                                                                        |
| **/api/workspaces/{workspace-id}/files/object/metadata**             | GET metadata for a single object store file                                                                                                                       |
| **/api/workspaces/{workspace-id}/users**                             | GET to list users, PUT or PATCH to change the list                                                                                                                |
| **/api/workspaces/{workspace-id}/users/{user-id}**                   | GET to fetch user information, PUT to create or change machine users for the workspace, PATCH to change machine users, DELETE to remove users from the workspace. |
| **/api/workspaces/{workspace-id}/{user-id\|'me'}/tokens**            | GET API tokens for workspace, POST to create                                                                                                                      |
| **/api/workspaces/{workspace-id}/{user-id\|'me'}/tokens/{token-id}** | DELETE to invalidate tokens                                                                                                                                       |
| **/api/workspaces/{workspace-id}/{user-id\|'me'}/s3-tokens**         | POST to create S3 credentials (GET and DELETE not supported)                                                                                                      |
| **/api/workspaces/{workspace-id}/{user-id\|'me'}/sessions**          | POST to create workspace-scoped session credentials (access and refresh tokens)                                                                                   |
| **/api/workspaces/{workspace-id}/accounting/usage-data**             | GET resource consumption data for a workspace (filterable by time range, paginated, supports `day`/`month` aggregation)                                           |
| ...                                                                  | More endpoints likely to be added                                                                                                                                 |

Access is controlled so that:
* For user-scoped tokens:
	* There is read-write access to all accounts endpoints for accounts owned by the user.
	* For workspaces inside an account owned by the user:
		* There is read-write access to `/api/workspaces`, `/api/workspaces/{workspace-id}` and `/api/workspaces/{workspace-id}/users`.
		* There is access to GET and DELETE `/api/workspaces/{workspace-id}/users/{user-id}` and to PUT or PATCH if `user-id` is a workspace machine user.
		* There is access to GET and DELETE `/api/workspaces/{workspace-id}/{user-id}/tokens` and specific tokens.
		* There is no access to create tokens.
	* For workspaces the user is a member of:
		* There is read-only access to`/api/workspaces`, `/api/workspaces/{workspace-id}`, `/api/workspaces/{workspace-id}/users` and `/api/workspaces/{workspace-id}/users/{user-id}`. If `user-id` is the calling user or the string `me` then there is also DELETE access.
		* Where `user-id` is a workspace machine user there is also write access to `/api/workspaces/{workspace-id}/users/{user-id}` and its sub-paths. There's also access to create machine users.
		* There is read access to `/api/workspaces`, `/api/workspaces/{workspace-id}` and `/api/workspaces/{workspace-id}/users` and read-write access to the tokens endpoints for workspaces of which the user is a member.
		* There is read-write access to the user's own tokens and tokens for the workspaces' machine users.
* For workspace-scoped tokens:
	* There is no access to accounts endpoints.
	* There is read access to `/api/workspaces` but only the scoped-to workspace is returned, and there is read access to `/api/workspaces/{scoped-to-workspace}`
	* There is read-write access to `/api/workspaces/{workspace-token-scoped-to}/{user-token-authorized-by}/tokens/{token-id}` and to the corresponding `s3-tokens` endpoint.
	* There is no access to endpoints for other users, even machine users.

The intent is that account owners can manage who is in a workspace (including adding themselves), workspace members with user tokens can act 'on' the workspace and workspace-scoped tokens can operate 'in' the workspace.

### Workspace Store Access

#### HTTPS-Based Access

This allows access to files in workspace stores, which can be used to serve public data or HTML as well as for private files. These use an alternative hostname to avoid any scope for session fixation attacks in which a workspace's HTML page tries to set a cookie on a parent domain.

Files are served rooted at https://{workspace-name}.eodatahub-workspaces.org.uk/ (for the workspace default object store) and at https://{store-id}.{workspace-name}.eodatahub-workspaces.org.uk/ for other stores. This allows users to host a web app at the root of the hostname should they wish.

Callers can append the path of any file in the store and use GET to retrieve it, PUT to create/replace it and DELETE to delete it.

An authorized workspace-scoped token is always required for PUT and DELETE. For GET, a workspace-scoped token always allows access (unless further scoped to limit access). Authorization without one depends on a user-set access policy and may either not require a token or may require a token scoped to certain other workspaces.

Future version of EODH may also add other access types, such as SFTP and WebDAV via SFTPGo, but these would use another hostname.

#### S3 Access

S3 access to workspace stores is possible by obtaining an AWS temporary token from **/api/workspaces/{workspace-id}/{user-id|'me'}/s3-tokens** and connecting to an S3 access point listed in **/api/workspaces/{workspace-id}** as one of the workspace object stores. This allows full access and is only possible for users who are members of the workspace.

S3 access is also possible if a workspace member has configured the bucket, or part of it, to be public using an access policy. The workspace is responsible for communicating the bucket and access point names, for example via a catalogue entry.


### Accounting API - **/api/accounting/**

These endpoints expose the billing item catalogue and current pricing. They require authentication but are not workspace-scoped.

|                                    |                                                                              |
| ---------------------------------- | ---------------------------------------------------------------------------- |
| **/api/accounting/skus**           | GET list of all billing items (SKUs), e.g. `wfcpu` (Workflow CPU seconds)   |
| **/api/accounting/skus/{sku}**     | GET a single billing item by SKU                                             |
| **/api/accounting/prices**         | GET all current prices (Pounds per unit, current period only)                |

Usage data is available per-workspace at `/api/workspaces/{workspace-id}/accounting/usage-data` and per-account at `/api/accounts/{account-id}/accounting/usage-data` (see tables above). Both endpoints support `start`/`end` time filtering, `limit`/`after` pagination, and a `time-aggregation` parameter (`day` or `month`).

### Workflow Catalogue - **/api/wf-catalogue/**

A separate OGC API Records-compliant catalogue for browsing and registering workflow and notebook records. This is distinct from the main STAC catalogue.

|                                                                  |                                                                                                |
| ---------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| **/api/wf-catalogue/collections**                                | GET list of workflow catalogues; POST to create a new catalogue (requires Bearer token)        |
| **/api/wf-catalogue/collections/{catalogue-id}**                 | GET catalogue metadata; DELETE to remove (requires Bearer token)                               |
| **/api/wf-catalogue/collections/{catalogue-id}/items**           | GET records in a catalogue (supports free-text `q`, `type` filter, pagination, `order_by`)    |
| **/api/wf-catalogue/collections/{catalogue-id}/items/{record-id}** | GET a single workflow or notebook record                                                     |
| **/api/wf-catalogue/register**                                   | POST to register a new workflow or notebook record (requires Bearer token; optional `?catalogue_id=`) |
| **/api/wf-catalogue/register/{record-id}**                       | DELETE a workflow or notebook record (requires Bearer token)                                   |

Record `type` is either `workflow` or `notebook`. Records follow the OGC Records / GeoJSON Feature structure.

### Visualization and Mapping - **/titiler/**

TiTiler provides raster visualization endpoints rooted at `/titiler/`. These endpoints do not require authentication and accept a `?url=` query parameter pointing to the source dataset.

#### COG / GeoTiff - **/titiler/core/cog/**

|                                                                                    |                                                          |
| ---------------------------------------------------------------------------------- | -------------------------------------------------------- |
| **/titiler/core/cog/bounds**                                                       | Dataset bounds                                           |
| **/titiler/core/cog/info**, **/titiler/core/cog/info.geojson**                     | Dataset info (JSON or GeoJSON)                           |
| **/titiler/core/cog/statistics**                                                   | GET band statistics; POST with GeoJSON geometry          |
| **/titiler/core/cog/tiles/{tileMatrixSetId}/{z}/{x}/{y}**                          | Map tile (multiple URL variants for scale and format)    |
| **/titiler/core/cog/{tileMatrixSetId}/tilejson.json**                              | Mapbox TileJSON                                          |
| **/titiler/core/cog/{tileMatrixSetId}/WMTSCapabilities.xml**                       | OGC WMTS GetCapabilities                                 |
| **/titiler/core/cog/{tileMatrixSetId}/map**                                        | HTML map viewer                                          |
| **/titiler/core/cog/preview**, **/titiler/core/cog/preview.{format}**              | Dataset preview image                                    |
| **/titiler/core/cog/bbox/{minx},{miny},{maxx},{maxy}.{format}**                    | Image from bounding box                                  |
| **/titiler/core/cog/feature**                                                      | POST GeoJSON Feature to get image                        |
| **/titiler/core/cog/point/{lon},{lat}**                                            | Pixel value at a point                                   |
| **/titiler/core/cog/validate**                                                     | Validate a COG                                           |
| **/titiler/core/cog/stac**                                                         | Generate a STAC Item from a COG                          |

#### Multidimensional Data (Zarr / xarray) - **/titiler/xarray/**

Requires `?url=` and `?variable=` query parameters. Supports `?datetime=`, `?group=`, `?drop_dim=`.

|                                                                   |                                                        |
| ----------------------------------------------------------------- | ------------------------------------------------------ |
| **/titiler/xarray/bounds**, **/titiler/xarray/info**              | Dataset bounds and info                                |
| **/titiler/xarray/variables**                                     | List available variables in a dataset                  |
| **/titiler/xarray/tiles/{tileMatrixSetId}/{z}/{x}/{y}**           | Map tile                                               |
| **/titiler/xarray/{tileMatrixSetId}/tilejson.json**               | Mapbox TileJSON                                        |
| **/titiler/xarray/{tileMatrixSetId}/WMTSCapabilities.xml**        | OGC WMTS GetCapabilities                               |
| **/titiler/xarray/statistics**                                    | POST GeoJSON to get zonal statistics                   |
| **/titiler/xarray/point/{lon},{lat}**                             | Pixel value at a point                                 |
| **/titiler/xarray/bbox/{minx},{miny},{maxx},{maxy}.{format}**     | Image from bounding box                                |

#### Tile Matrix Sets, Algorithms and Colormaps

|                                              |                                                                      |
| -------------------------------------------- | -------------------------------------------------------------------- |
| **/titiler/tileMatrixSets**                  | List available tile matrix sets (WebMercatorQuad, WorldCRS84Quad, etc.) |
| **/titiler/tileMatrixSets/{id}**             | Definition of a specific tile matrix set                             |
| **/titiler/algorithms**                      | List available algorithms (hillshade, slope, normalizedIndex, etc.)  |
| **/titiler/algorithms/{algorithmId}**        | Metadata for a specific algorithm                                    |
| **/titiler/colorMaps**                       | List available colormaps                                             |
| **/titiler/colorMaps/{colorMapId}**          | Colormap data or preview image                                       |

#### WMTS via Catalogue

The catalogue-integrated WMTS endpoint is also available at `/titiler/{catalog_path}/wmts`, supporting OGC WMTS `GetCapabilities`, `GetTile`, and `GetFeatureInfo` operations via query parameters.

### EODH Authorization Endpoints

Unlike the endpoints above, these **are not rooted under /api**. Instead, clients should look up https://eodatahub.org.uk/.well-known/openid-configuration which will provide links to OIDC and OAuth endpoints.

These endpoints will be under https://eodatahub.org.uk/keycloak/realms/eodhp/:
* **protocol/openid-connect/auth**: OIDC authorization endpoint
* **protocol/openid-connect/token**: OIDC token endpoint
* **protocol/openid-connect/userinfo**: OIDC userinfo endpoint
* **protocol/openid-connect/logout**: Keycloak logout endpoint
* **protocol/openid-connect/token/introspect**: OAuth token introspection endpoint
* **protocol/openid-connect/revoke**: OAuth token revocation endpoint
