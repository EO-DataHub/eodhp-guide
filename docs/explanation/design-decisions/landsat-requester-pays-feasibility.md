---
title: "Feasibility: Landsat from Earth Search and requester-pays access"
doc_status: unreviewed
tags:
  - stac
  - data-catalogues
  - aws
  - workspaces
last_reviewed:
reviewed_by:
review_notes: "Feasibility study for eodhp-sparkgeo-pm#246, updated 2026-11-02 with the proof-of-concept results."
---
# Feasibility: Landsat from Earth Search and requester-pays access

**Status:** Feasibility study with proof-of-concept results. The approach works. No decision is made yet.
**Date:** 2026-11-02
**Applies to:** [eodhp-sparkgeo-pm#246](https://github.com/EO-DataHub/eodhp-sparkgeo-pm/issues/246). It shares a mechanism with download tracking, [feature-requests#16](https://github.com/EO-DataHub/feature-requests/issues/16).

## Summary

Landsat Collection 2 Level-2 data can be made available through the Hub, but the plain redirect proposed for download tracking does not work for it. The Landsat data is in a requester-pays S3 bucket. So the Hub must redirect to an S3 URL that it has signed itself, and it pays AWS for each read. The Hub then needs a reliable record of bytes read per workspace to recharge the cost.

Most of the catalogue side exists already. A proof of concept (PoC) tested signing, metering and client behaviour against the real `usgs-landsat` bucket. Its results:

- **The redirect works.** curl, GDAL/rasterio, QGIS, pystac-client, a notebook, ADES stage-in and a UI download all read Landsat through the endpoint. GDAL reads take the same time as direct access.
- **CloudTrail meters the reads per workspace.** The reads appear in the Hub's own trail, named by the workspace's role session, with exact byte counts, about 5 minutes after the read.
- **Browser JavaScript cannot read the data.** USGS has CORS turned off on the bucket. Downloads by link work. In-browser COG viewers cannot work through a redirect.
- **Titiler must check access itself.** GDAL's cache, inside one titiler process, serves a second user without asking the endpoint.

See [Proof of concept](#proof-of-concept) for the details.

## Context

### What exists today

| Part | State |
|---|---|
| Catalogue proxy | The Earth Search proxy is deployed in `eodhp-argocd-deployment` at `apps/resource-catalogue/base/proxies/element84/`. It serves Earth Search items and search through the Hub domain, and leaves asset hrefs unchanged. |
| Harvester | `element84-harvester` writes catalog and collection documents only, for the collections in `VALID_COLLECTIONS` (now `sentinel-2-c1-l2a`). Item search goes through the proxy. To show Landsat, add `landsat-c2-l2` to that list. See [Repositories](../../reference/repositories.md). |
| Billing | `BillingEvent` in `eodhp-utils` and the egress SKUs in `eodhp-data-transfer-events` (`EGRESS-REGION`, `EGRESS-INTERREGION`, `EGRESS-INTERNET`) already price data transfer per workspace. A Landsat SKU fits this model. |
| Redirect or presign endpoint | None exists in any Hub service. |

### The Landsat assets

In Earth Search, each `landsat-c2-l2` data asset has an `s3://usgs-landsat/...` href. The asset carries `storage:requester_pays: true` and `storage:region: us-west-2`. Browsers and QGIS cannot open `s3://` URLs, so the Hub must rewrite these hrefs whichever option it chooses.

The local test data in `element84-data` has `s3:/usgs-landsat/...` hrefs, with one slash. This is probably an error in how the data was saved. Tests for href rewriting must not depend on it.

## Problems that requester-pays adds

### A plain redirect fails

An unsigned GET to `usgs-landsat` returns 403. The redirect must point at a **presigned S3 URL**. An IAM principal of the Hub signs it, and the URL carries `x-amz-request-payer=requester`. Presigning is a local computation with no call to AWS, so it adds almost no latency. The Hub's AWS account pays AWS for each read, and the Hub recharges the workspace.

boto3 does not produce this URL by default. Given `RequestPayer='requester'`, it signs `x-amz-request-payer` as a header (`X-Amz-SignedHeaders=host;x-amz-request-payer`). A client that follows a redirect does not send that header, so S3 rejects the signature. The signer must move the value into the query string before signing. The PoC does this with a botocore `before-sign` hook, and S3 accepts the result. The hook must act on presigned requests only. On an ordinary header-signed call, S3 refuses the query-string form with 403. Element 84 uses the same query-string form for its own Earth Search thumbnail redirects to `usgs-landsat`.

A presigned URL is valid for one HTTP method, and GDAL reuses the target of its first redirect for later range GETs. If the endpoint redirects a HEAD, GDAL sends its GETs to a HEAD-signed URL and S3 refuses them. So **the endpoint answers HEAD itself**, with a server-side `HeadObject` call as the workspace's session, and redirects only GET. CloudTrail still attributes the `HeadObject`.

### The redirect endpoint cannot measure volume

Bytes go directly from S3 to the client, so the endpoint never sees them. Also, GDAL caches the target of a redirect to a presigned S3 URL and sends its range reads directly to S3. In the PoC, a GDAL window and overview read made 2 requests to the endpoint (HEAD and GET) and 5 range reads to S3. This is good for performance, but the endpoint sees only 2 requests per COG, so the range-logging plan in feature-requests#16 cannot measure volume.

### Every data request must name a workspace

The endpoint must know which workspace to charge, so a data request must carry a token. Catalogue browsing stays public, but anonymous reads of this data stop. This is a change for users, and the user documentation must explain it.

A token can list more than one workspace (the `workspaces` claim), and the asset hrefs are the same for every user. So the endpoint chooses the workspace in this order:

1. The `X-EODH-Workspace` header
2. The `workspace` query parameter
3. The token's only workspace, if it lists exactly one

If none of these applies, the endpoint returns 400. The chosen workspace must be in the token's `workspaces` claim, or the endpoint returns 403. The query parameter is as safe as the header, because the endpoint checks both against the same claim. Clients that cannot send a header need it: a link click, and a tile URL.

| Access path | How it authenticates | How it names the workspace |
|---|---|---|
| GDAL, QGIS, `pyeodh` | Bearer token (`GDAL_HTTP_HEADERS`, or a QGIS **API Header** auth configuration whose value is `Bearer <token>`) | Header. Not the query parameter: see below. |
| Notebook | Workspace access token | The token's only workspace (tested with a local token; still to confirm that real workspace tokens list one workspace) |
| ADES stage-in | `ades-stagein/stage.py` sends `WORKSPACE_ACCESS_TOKEN` as a bearer token with `requests.get` | The token's only workspace |
| UI download (link click) | A link click sends no bearer header, so the endpoint sits behind the auth agent, which authenticates the session cookie | `?workspace=` from the UI's active workspace |
| Titiler tiles | Titiler forwards the user's token to the endpoint (see [Titiler](#titiler)) | `?workspace=` on the URL that titiler gets |

GDAL clients must name the workspace with the header. GDAL probes for sidecar files (`.aux.xml`, `.xml`, `_rpc.txt` and others) by changing the URL. It appends the extension after the query string (`?workspace=ws-a.aux.xml`), or swaps the extension in the path and drops the query. The endpoint rejects every such probe. Each probe is a wasted request, but none reaches S3. Clients must also set `GDAL_DISABLE_READDIR_ON_OPEN=EMPTY_DIR`, which stops the probing. The `eodh-qgis` plugin and `pyeodh` can set both for the user.

GDAL caches a failed read for the rest of the process. After a 401, QGIS does not ask for the file again until it restarts or `gdal.VSICurlClearCache()` runs.

### A client must not send its token to S3

S3 rejects a request that carries both an `Authorization` header and a query-string signature (400, `Only one auth mechanism allowed`). A client that sends the Hub token to the endpoint must drop it when it follows the redirect to S3. In the PoC, curl, `requests` and GDAL 3.12 all dropped it. But GDAL sends `GDAL_HTTP_HEADERS` to every host when it opens a URL itself, so a user who opens a presigned S3 URL directly with the token still set gets 400. GDAL's path-specific options (a `[credentials]` section in a GDAL config file) send the token only to the Hub's URL prefix. This is the safer setup for `pyeodh` and the QGIS plugin.

## Titiler

If titiler reads `usgs-landsat` with platform credentials, anyone can read the data through tiles at the platform's cost. So titiler must read through the endpoint, like every other client:

1. The UI gives titiler the rewritten asset URL, with `?workspace=`, in place of the `s3://` href.
2. Titiler already reads the JWT and checks the `workspaces` claim. It passes the user's token to GDAL as a header for that URL.
3. GDAL follows the redirect, and the read is charged to the workspace.

Titiler's own role assumption cannot attribute the cost. It uses the fixed session name `titiler-core` (`titiler/core/auth.py`), so every read looks the same in CloudTrail.

One risk is specific to a shared server. GDAL caches the target of a redirect per URL, inside one process. Titiler serves many users from one process, so a second user can reuse the presigned URL from the first user's request. The PoC confirmed this. One process read the same COG for three callers in turn:

| Caller | Same URL for every caller | `?workspace=` in the URL |
|---|---|---|
| User 1, charging ws-a | Endpoint, charged to ws-a | Endpoint, charged to ws-a |
| User 1, charging ws-b | **No endpoint request. Charged to ws-a.** | Endpoint, charged to ws-b |
| User 2, naming ws-a without access | **Read succeeds from the cache** | **Read succeeds from the cache** |

`?workspace=` puts the charge on the right workspace, but it does not stop an unauthorised read. **Titiler must check the workspace claim on each request, before GDAL reads.** It must not rely on the endpoint for that check.

Tile requests for Landsat then need a login, the same as downloads.

## Metering options

| Option | Accuracy | Cost and effort | Assessment |
|---|---|---|---|
| CloudTrail S3 data events in the Hub's account | Exact. Each `GetObject` event records `bytesTransferredOut`. | About $0.10 per 100,000 events. COG range reads make many events. | **Preferred. Confirmed by the PoC.** |
| Charge the asset's `file:size` on the first request | Overcharges partial COG reads | Small | Fallback |
| Proxy the bytes through the Hub | Exact | About twice the transfer cost (us-west-2 to eu-west-2 to the internet), plus compute | Not recommended |

The CloudTrail option has one more advantage. If the Hub signs with a role session named after the workspace, each event identifies the workspace. The same record then covers all access paths: redirected downloads, and notebooks and workflows that read S3 directly with the workspace role.

The PoC confirmed that this works. A trail in the requester's account, with one advanced event selector (`eventCategory=Data`, `resources.type=AWS::S3::Object`, `resources.ARN` starting with `arn:aws:s3:::usgs-landsat/`, read-only), records each read:

- The event reaches the requester's trail (`recipientAccountId` is the Hub's account). USGS gets its own copy.
- `userIdentity.arn` ends with the session name, `.../requester-pays-poc-signer/<workspace>`.
- `bytesTransferredOut` matched every client. Over one run, each file and workspace total equalled what the clients read.
- A request that S3 refuses (bad signature, wrong method, two auth mechanisms) leaves no event.
- The first log file arrived about 5 minutes after the read.

Two cost effects are visible only in CloudTrail, which supports it as the metering source:

- **AWS charges for bytes the client does not use.** GDAL's first, unranged GET sends 1.1 MB, though GDAL reads only the header and closes the connection early.
- **AWS charges for requests that CORS blocks.** The browser sends the request, and S3 sends the data. The browser only hides the response from the page.

`lookup-events` does not return data events. The emitter must read the log files that the trail delivers to its S3 bucket. Athena can query the same files for reports.

## Cost estimate

These are AWS list prices. Check them before quoting.

| Access path | Transfer cost |
|---|---|
| Download to the internet | About $0.09 per GB |
| Read from the Hub cluster: inter-region transfer | About $0.02 per GB |
| Read from the Hub cluster: NAT gateway processing | About $0.045 per GB |
| GET requests | About $0.0004 per 1,000 |

A read from the Hub cluster costs about $0.065 per GB in total. This covers notebooks, workflows, ADES stage-in and titiler. A gateway VPC endpoint for S3 serves only buckets in the same region, so traffic to `usgs-landsat` in us-west-2 goes through a NAT gateway. See also [Credit rate derivation](credit-rate-derivation.md), which notes that NAT traffic is not metered today.

A full Level-2 scene is about 1 GB.

## Other risks

- **Browser COG viewers cannot work.** CORS is not enabled on `usgs-landsat` (`CORSResponse: CORS is not enabled for this bucket`), and USGS owns the bucket. JavaScript in a browser (`fetch`, in-browser COG libraries) cannot read the data through any redirect. Navigation (links, downloads, new tabs) is not subject to CORS and works. The Hub's own UIs do not read COGs in the browser: their maps go through titiler. The `stac-browser` fork can, but no deployment of it was found. A browser viewer for this data would need the bytes proxied through the Hub.
- **Thumbnails are not free.** The `thumbnail` and `reduced_resolution_browse` assets are in `usgs-landsat` and are rewritten like any other asset, so they would need a login and cost a workspace each time a page shows them. The item's `rel=thumbnail` link is not a free alternative. Earth Search answers it with a 302 to a URL presigned with Element 84's own credentials, so Element 84 pay for each view. That is a courtesy with no SLA, and the Hub should not depend on it. This needs a decision between two options:
    - **Sign thumbnails through the Hub and pay for them.** `thumb_small.jpeg` files are small, so the cost may be negligible. Every catalogue page that shows Landsat thumbnails would need a login. The Hub could also carry the cost itself, not charge it to a workspace.
    - **Use Planetary Computer's rendered preview.** It needs no login. Its `landsat-c2-l2` item IDs match Earth Search's, so the UI can build the URL from `item.id`: `https://planetarycomputer.microsoft.com/api/data/v1/item/preview.png?collection=landsat-c2-l2&item={id}&assets=red&assets=green&assets=blue&color_formula=gamma%20RGB%202.7,%20saturation%201.5,%20sigmoidal%20RGB%2015%200.55&format=png`. A test on 2026-11-03 returned a 67 KB PNG in about 1.8 s. It is a third-party service with no SLA, it renders on request, and the image looks a little different from the USGS thumbnail.
- **`storage:*` metadata no longer matches the href.** The rewritten href points at the Hub, but `storage:platform`, `storage:region` and `storage:requester_pays` still describe S3. pystac warns about this. Decide whether to drop or keep these fields on rewrite.
- **Render configurations must use the rewritten URLs.** See [Titiler](#titiler) and [Render configurations](../architecture/render-configs.md).

## Proposed URL scheme

`GET` or `HEAD /requester-pays/{bucket}/{key}[?workspace=<name>]`, with an allowlist of buckets. At first the allowlist contains `usgs-landsat` only. GET returns a 307 redirect to a presigned GET URL. HEAD returns 200 with `Content-Length`, `Content-Type`, `ETag`, `Last-Modified` and `Accept-Ranges: bytes`, from a server-side `HeadObject`.

The endpoint builds the redirect target from the bucket and the key, so it cannot become an open redirect, and it does not need to look up the item. The scheme also makes href rewriting one `sub_filter` line in the existing nginx proxy:

```nginx
sub_filter 's3://usgs-landsat/' 'https://<platform-domain>/requester-pays/usgs-landsat/';
```

Feature-requests#16 proposes a scheme based on item and asset IDs. The two tickets should share one mechanism, so the owners of both must agree on the scheme.

## Proof of concept

The PoC ran on 2026-11-02 in the EODH-test AWS account, from a laptop on home broadband. It is a separate project (`requester-pays-poc`, not yet published) and changes no existing repository. It uses local RS256 tokens with the platform's claim names, in place of Keycloak.

### Results

All tests used one Landsat scene, `LC09_L2SR_089113_20261101_02_T2`.

| Client | Result | Requests to the endpoint | Token sent to S3 |
|---|---|---|---|
| curl, MTL file (16 KB) | Works. Time to first byte 0.70 s, compared with 0.60 s direct. | 1 | No |
| curl, SR_B4 band (71 MB) | Works. 35.4 s, compared with 36.7 s direct. | 1 | No |
| GDAL/rasterio, window and overview read | Works. Same pixels as a direct read. 3.6 s, compared with 3.4 s direct. | 2 (HEAD, GET) | No |
| pystac-client through the proxy | All 16 asset hrefs rewritten | — | — |
| QGIS | Works, with an API Header auth configuration of `Bearer <token>` | 2, plus rejected sidecar probes | No |
| Notebook (single-workspace token, no header) | Works. Charged to the token's workspace. | 2 | No |
| ADES stage-in (`requests.get` with a bearer token) | Works without change | 1 | No |
| UI download (link click, cookie through an emulated auth agent) | Works | 1 | No |
| Browser `fetch` with Range | **Fails: CORS** (see [Other risks](#other-risks)) | 1 | No |
| Titiler (simulated) | **GDAL's cache bypasses the endpoint** (see [Titiler](#titiler)) | 1 per URL | No |

The extra round trip to the endpoint is within the noise of these measurements.

### Open questions

- Do real workspace access tokens list exactly one workspace? The notebook and ADES paths depend on it.
- How many data events, and what trail cost, does a realistic COG session produce? The PoC reads were small.
- How does the endpoint run behind the real auth agent? The PoC emulated it with nginx.
- Which GDAL versions do users run? The redirect behaviour above is from GDAL 3.12.2.
- `pyeodh` was not tested. It reads assets with the same HTTP stack as pystac-client, but it must send the workspace header.
- Next step: a `BillingEvent` emitter that reads the trail's log files and totals `bytesTransferredOut` per workspace.

### Plan

The plan the PoC followed, kept for reference.

1. Write a small FastAPI endpoint. It does these things:
    - It validates the bearer token and chooses the workspace, as in [Every data request must name a workspace](#every-data-request-must-name-a-workspace).
    - It assumes a role with `RoleSessionName=<workspace>`.
    - It presigns the GET with `RequestPayer='requester'`, in the query string, and a short expiry. It answers HEAD itself.
    - It returns a 307 redirect and writes a structured log line.
2. Run a local copy of the Earth Search proxy with the extra `sub_filter` line, pointed at the endpoint. The same nginx emulates the auth agent for the UI download test: it turns a session cookie into a bearer token.
3. Test the clients in the table below.
4. Turn on a trail for S3 data events in the sandbox account.

| Client | What the test does | What it shows |
|---|---|---|
| curl | Full GET through the redirect | The presigned URL works against S3. Volume and latency compared with direct access. |
| rasterio/GDAL | Window read and overview read, token in `GDAL_HTTP_HEADERS` | Redirect caching. Whether GDAL sends the token to S3 too, which S3 rejects. |
| pystac-client and `pyeodh` | Search through the proxy, then read an asset href | The rewritten hrefs work from a STAC client |
| QGIS (manual) | Add a rewritten COG URL as a layer, with a headers auth configuration | Desktop GIS works. Pan and zoom behaviour. |
| Notebook | rasterio with a token that lists one workspace, no header | The workspace token path |
| ADES stage-in | `requests.get` with a bearer token, as `ades-stagein/stage.py` does | Stage-in works without change |
| UI download | Browser link click with `?workspace=`, through the emulated auth agent | A link click works without a bearer header |
| Browser COG viewer | `fetch` with a Range header | CORS on the S3 response |
| Titiler (simulated) | One long-lived process reads the same COG for two users in different workspaces | Whether GDAL's redirect cache crosses users or workspaces |

#### What to measure

For each client:

- The number of requests that reach the endpoint, and the number that reach S3
- Time to first byte and total read time, compared with direct presigned access
- Whether the client works at all (CORS, how it sends the token and the workspace)

For metering:

- Whether the data events arrive in the sandbox trail
- Whether each event carries the role session name
- Whether `bytesTransferredOut` matches the bytes that the clients read

If the metering checks pass, the next step is a `BillingEvent` emitter that reads the trail.

#### Prerequisites

- A sandbox AWS account with an IAM role that allows `s3:GetObject` on `arn:aws:s3:::usgs-landsat/*`
- FastAPI, boto3 and PyJWT in the scratch project

## Tasks in #246 that this study covers

| Task in #246 | Where this page covers it |
|---|---|
| PoC of the redirect or proxy mechanism | [Proof of concept](#proof-of-concept): done, see [Results](#results) |
| Harvest or link to Earth Search | [What exists today](#what-exists-today): harvest collections, serve items through the proxy |
| Charge requester-pays costs through `accounting-service` | [Metering options](#metering-options) |
| Check access from RC UI, QGIS, `pyeodh`, notebooks | [Every data request must name a workspace](#every-data-request-must-name-a-workspace), and the client table in [Results](#results) |
| Render configuration | [Titiler](#titiler) |
