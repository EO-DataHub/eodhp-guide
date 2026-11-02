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
review_notes: "Initial feasibility study for eodhp-sparkgeo-pm#246. Claims marked 'to confirm' are for the proof of concept to test."
---
# Feasibility: Landsat from Earth Search and requester-pays access

**Status:** Feasibility study. No decision is made yet. The proof of concept (PoC) below will supply the evidence for one.
**Date:** 2026-11-02
**Applies to:** [eodhp-sparkgeo-pm#246](https://github.com/EO-DataHub/eodhp-sparkgeo-pm/issues/246). It shares a mechanism with download tracking, [feature-requests#16](https://github.com/EO-DataHub/feature-requests/issues/16).

## Summary

Landsat Collection 2 Level-2 data can be made available through the Hub, but the plain redirect proposed for download tracking does not work for it. The Landsat data is in a requester-pays S3 bucket. So the Hub must redirect to an S3 URL that it has signed itself, and it pays AWS for each read. The Hub then needs a reliable record of bytes read per workspace to recharge the cost.

Most of the catalogue side exists already. The open questions are about signing, metering, and client behaviour. A PoC of about two to three days can answer them.

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

boto3 does not produce this URL by default. Given `RequestPayer='requester'`, it signs `x-amz-request-payer` as a header (`X-Amz-SignedHeaders=host;x-amz-request-payer`). A client that follows a redirect does not send that header, so S3 rejects the signature. The signer must move the value into the query string before signing. The PoC does this with a botocore `before-sign` hook (tested offline; to confirm against S3).

A presigned URL is valid for one HTTP method. Clients can send HEAD before GET, so the endpoint signs each method separately.

### The redirect endpoint cannot measure volume

Bytes go directly from S3 to the client, so the endpoint never sees them. Also, GDAL is expected to cache the target of a redirect to a presigned S3 URL and send its range reads directly to S3 (`CPL_VSIL_CURL_USE_S3_REDIRECT`, on by default; to confirm). This is good for performance, because a client makes about one extra round trip per file. But the endpoint may see only one request per COG, so the range-logging plan in feature-requests#16 cannot measure volume.

### Every data request must name a workspace

The endpoint must know which workspace to charge, so a data request must carry a token. Catalogue browsing stays public, but anonymous reads of this data stop. This is a change for users, and the user documentation must explain it.

A token can list more than one workspace (the `workspaces` claim), and the asset hrefs are the same for every user. So the endpoint chooses the workspace in this order:

1. The `X-EODH-Workspace` header
2. The `workspace` query parameter
3. The token's only workspace, if it lists exactly one

If none of these applies, the endpoint returns 400. The chosen workspace must be in the token's `workspaces` claim, or the endpoint returns 403. The query parameter is as safe as the header, because the endpoint checks both against the same claim. Clients that cannot send a header need it: a link click, and a tile URL.

| Access path | How it authenticates | How it names the workspace |
|---|---|---|
| GDAL, QGIS, `pyeodh` | Bearer token (`GDAL_HTTP_HEADERS`, a QGIS auth configuration with headers) | Header |
| Notebook | Workspace access token | The token's only workspace (to confirm that workspace tokens list one workspace) |
| ADES stage-in | `ades-stagein/stage.py` sends `WORKSPACE_ACCESS_TOKEN` as a bearer token with `requests.get` | The token's only workspace |
| UI download (link click) | A link click sends no bearer header, so the endpoint sits behind the auth agent, which authenticates the session cookie | `?workspace=` from the UI's active workspace |
| Titiler tiles | Titiler forwards the user's token to the endpoint (see [Titiler](#titiler)) | `?workspace=` on the URL that titiler gets |

### A client must not send its token to S3

S3 rejects a request that carries both an `Authorization` header and a query-string signature. A client that sends the Hub token to the endpoint must drop it when it follows the redirect to S3. curl and `requests` drop `Authorization` when the host changes. GDAL with `GDAL_HTTP_HEADERS` is not yet known. The PoC tests it.

## Titiler

If titiler reads `usgs-landsat` with platform credentials, anyone can read the data through tiles at the platform's cost. So titiler must read through the endpoint, like every other client:

1. The UI gives titiler the rewritten asset URL, with `?workspace=`, in place of the `s3://` href.
2. Titiler already reads the JWT and checks the `workspaces` claim. It passes the user's token to GDAL as a header for that URL.
3. GDAL follows the redirect, and the read is charged to the workspace.

Titiler's own role assumption cannot attribute the cost. It uses the fixed session name `titiler-core` (`titiler/core/auth.py`), so every read looks the same in CloudTrail.

One risk is specific to a shared server. GDAL caches the target of a redirect per URL, inside one process. Titiler serves many users from one process, so a second user could reuse the presigned URL from the first user's request. That read skips the endpoint and is charged to the first user's workspace. `?workspace=` keeps different workspaces' URLs apart in the cache. Titiler must still check the workspace claim on each request, before GDAL reads, because the cache does not check it. The PoC tests this behaviour in one process.

Tile requests for Landsat then need a login, the same as downloads.

## Metering options

| Option | Accuracy | Cost and effort | Assessment |
|---|---|---|---|
| CloudTrail S3 data events in the Hub's account | Exact. Each `GetObject` event records `bytesTransferredOut`. | About $0.10 per 100,000 events. COG range reads make many events. | **Preferred, if the PoC confirms it.** |
| Charge the asset's `file:size` on the first request | Overcharges partial COG reads | Small | Fallback |
| Proxy the bytes through the Hub | Exact | About twice the transfer cost (us-west-2 to eu-west-2 to the internet), plus compute | Not recommended |

The CloudTrail option has one more advantage. If the Hub signs with a role session named after the workspace, each event identifies the workspace. The same record then covers all access paths: redirected downloads, and notebooks and workflows that read S3 directly with the workspace role.

**This depends on one unverified fact.** We do not yet know if S3 data events for a bucket in another account reach the requester's trail. If they do not, the CloudTrail option fails. This is the most important question for the PoC.

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

- **Browser COG viewers may fail.** USGS owns the `usgs-landsat` bucket and controls its CORS policy. The PoC must test a browser `fetch`.
- **Render configurations must use the rewritten URLs.** See [Titiler](#titiler) and [Render configurations](../architecture/render-configs.md).

## Proposed URL scheme

`GET` or `HEAD /requester-pays/{bucket}/{key}[?workspace=<name>]`, with an allowlist of buckets. At first the allowlist contains `usgs-landsat` only.

The endpoint builds the redirect target from the bucket and the key, so it cannot become an open redirect, and it does not need to look up the item. The scheme also makes href rewriting one `sub_filter` line in the existing nginx proxy:

```nginx
sub_filter 's3://usgs-landsat/' 'https://<platform-domain>/requester-pays/usgs-landsat/';
```

Feature-requests#16 proposes a scheme based on item and asset IDs. The two tickets should share one mechanism, so the owners of both must agree on the scheme.

## Proof of concept

The PoC is in a new scratch project and does not change the existing repositories. It takes about two to three days.

### Steps

1. Write a small FastAPI endpoint. It does these things:
    - It validates the bearer token and chooses the workspace, as in [Every data request must name a workspace](#every-data-request-must-name-a-workspace).
    - It assumes a role with `RoleSessionName=<workspace>`.
    - It presigns the GET or HEAD with `RequestPayer='requester'`, in the query string, and a short expiry.
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

### What to measure

For each client:

- The number of requests that reach the endpoint, and the number that reach S3
- Time to first byte and total read time, compared with direct presigned access
- Whether the client works at all (CORS, how it sends the token and the workspace)

For metering:

- Whether the data events arrive in the sandbox trail
- Whether each event carries the role session name
- Whether `bytesTransferredOut` matches the bytes that the clients read

If the metering checks pass, the next step is a `BillingEvent` emitter that reads the trail.

### Prerequisites

- A sandbox AWS account with an IAM role that allows `s3:GetObject` on `arn:aws:s3:::usgs-landsat/*`
- FastAPI, boto3 and PyJWT in the scratch project

## Tasks in #246 that this study covers

| Task in #246 | Where this page covers it |
|---|---|
| PoC of the redirect or proxy mechanism | [Proof of concept](#proof-of-concept) |
| Harvest or link to Earth Search | [What exists today](#what-exists-today): harvest collections, serve items through the proxy |
| Charge requester-pays costs through `accounting-service` | [Metering options](#metering-options) |
| Check access from RC UI, QGIS, `pyeodh`, notebooks | [Every data request must name a workspace](#every-data-request-must-name-a-workspace), and the client table in [Steps](#steps) |
| Render configuration | [Titiler](#titiler) |
