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

### The redirect endpoint cannot measure volume

Bytes go directly from S3 to the client, so the endpoint never sees them. Also, GDAL is expected to cache the target of a redirect to a presigned S3 URL and send its range reads directly to S3 (`CPL_VSIL_CURL_USE_S3_REDIRECT`, on by default; to confirm). This is good for performance, because a client makes about one extra round trip per file. But the endpoint may see only one request per COG, so the range-logging plan in feature-requests#16 cannot measure volume.

### Anonymous access stops for this data

The endpoint must know which workspace to charge, so a data request must carry a token. Catalogue browsing stays public. GDAL and QGIS can send a bearer token (`GDAL_HTTP_HEADERS`), and `pyeodh` can add it for the user. This is a change for users, and the user documentation must explain it.

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
| Read from the Hub cluster (inter-region) | About $0.02 per GB |
| GET requests | About $0.0004 per 1,000 |

A full Level-2 scene is about 1 GB.

## Other risks

- **Browser COG viewers may fail.** USGS owns the `usgs-landsat` bucket and controls its CORS policy. The PoC must test a browser `fetch`.
- **Rendering cost has no workspace.** Titiler needs `AWS_REQUEST_PAYER=requester` and platform credentials. The platform pays for rendering, so it must either absorb the cost or limit the rendering rate. See [Render configurations](../architecture/render-configs.md).

## Proposed URL scheme

`GET /requester-pays/{bucket}/{key}`, with an allowlist of buckets. At first the allowlist contains `usgs-landsat` only.

The endpoint builds the redirect target from the bucket and the key, so it cannot become an open redirect, and it does not need to look up the item. The scheme also makes href rewriting one `sub_filter` line in the existing nginx proxy:

```nginx
sub_filter 's3://usgs-landsat/' 'https://<platform-domain>/requester-pays/usgs-landsat/';
```

Feature-requests#16 proposes a scheme based on item and asset IDs. The two tickets should share one mechanism, so the owners of both must agree on the scheme.

## Proof of concept

The PoC is in a new scratch project and does not change the existing repositories. It takes about two to three days.

### Steps

1. Write a small FastAPI endpoint. It does these things:
    - It validates the bearer token and gets the workspace from it.
    - It assumes a role with `RoleSessionName=<workspace>`.
    - It presigns the GET with `RequestPayer='requester'` and a short expiry.
    - It returns a 307 redirect and writes a structured log line.
2. Run `element84-proxy` locally with the extra `sub_filter` line, pointed at the endpoint.
3. Test these clients: `curl`, rasterio/GDAL (full download and windowed read), QGIS, pystac-client and `pyeodh`, and a browser `fetch`.
4. Turn on a trail for S3 data events in the sandbox account.

### What to measure

For each client:

- The number of requests that reach the endpoint, and the number that reach S3
- Time to first byte and total read time, compared with direct presigned access
- Whether the client works at all (CORS, how it sends the token)

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
| Check access from RC UI, QGIS, `pyeodh`, notebooks | [Steps](#steps), step 3 |
| Render configuration | [Other risks](#other-risks): rendering cost |
