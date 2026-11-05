---
title: "ADR-003: URL contract for requester-pays data access"
doc_status: unreviewed
tags:
  - stac
  - data-catalogues
  - aws
  - workspaces
last_reviewed:
reviewed_by:
review_notes: "All six decisions made on 2026-11-04. PR #76 review by geodowd addressed 2026-11-05 (HEAD 403 mapping, lifetime vs file size). Acceptance conditions still open."
---
# ADR-003: URL contract for requester-pays data access

**Status:** Proposed. The decisions are made; acceptance waits on the conditions at the end.
**Date:** 2026-11-04
**Applies to:** [eodhp-sparkgeo-pm#246](https://github.com/EO-DataHub/eodhp-sparkgeo-pm/issues/246), tracked in [#259](https://github.com/EO-DataHub/eodhp-sparkgeo-pm/issues/259). It shares a mechanism with download tracking, [feature-requests#16](https://github.com/EO-DataHub/feature-requests/issues/16).

## Summary

A client reads an object in a requester-pays bucket at `/api/data/objects/{bucket}/{key}`, with a bearer token. The endpoint chooses the workspace to charge from the `X-EODH-Workspace` header, the `workspace` query parameter, or the token's only workspace. GET returns a 307 redirect to an S3 URL that the Hub signs, valid for 15 minutes by default. The lifetime limits when a download can start, not how long it can take, so file size does not matter. HEAD is answered by the endpoint itself. Catalogue preview images need no token, and the Hub pays for them.

## Context

Landsat Collection 2 Level-2 data is in the requester-pays S3 bucket `usgs-landsat`. To offer it, the Hub signs each read itself and pays AWS, and charges the reading workspace for it. A new Hub endpoint checks the user's token and redirects the client to a presigned S3 URL. The [feasibility study](landsat-requester-pays-feasibility.md) and its proof of concept showed that this works, and that CloudTrail meters each read per workspace.

This ADR fixes the URL contract of that endpoint: what a client requests, what it must send, and what it gets back. The contract must be settled first, because many consumers build on it:

| Consumer | How it uses the contract |
|---|---|
| Earth Search catalogue proxy | Rewrites `s3://usgs-landsat/...` hrefs to endpoint URLs with one `sub_filter` line |
| Web UI | Download links, and the thumbnails on catalogue pages |
| Titiler | Reads COGs through the endpoint for map tiles |
| `pyeodh`, the QGIS plugin, notebooks, ADES stage-in | Open endpoint URLs with GDAL or `requests` and a bearer token |
| Auth agent and OPAL policy | Decide which paths need a login |
| Download tracking (feature-requests#16) | Proposes its own redirect scheme. The two must share one mechanism |

The proof of concept found these constraints:

- **GDAL caches the redirect target** and sends its range reads straight to S3. The endpoint sees about two requests per file, not every read.
- **A presigned URL is valid for one HTTP method.** GDAL reuses the target of its first redirect, so a redirected HEAD breaks its later GETs.
- **S3 rejects a request that carries both a bearer token and a query-string signature.** Clients must not send the Hub token to S3.
- **GDAL probes for sidecar files by changing the URL.** It appends to the query string, so a query parameter is unreliable for GDAL clients.
- **CORS is disabled on `usgs-landsat`.** Browser JavaScript cannot read the data. Links, downloads and `<img>` tags work.
- **A link click sends a cookie, not a bearer token.** The endpoint must sit behind the auth agent for UI downloads.

What is known about tokens:

- Workspace-scoped tokens come from a Keycloak token exchange with scope `workspace:<id>` (`eodhp-workspace-services/api/handlers/sessions.go`). A real token of this kind lists one workspace in its `workspaces` claim, with a matching `workspace` claim, and `aud` includes `eodh`.
- Unknown: what the auth agent's session cookie carries for a link click, and what a workspace API token (`EODH...`) carries.

## Decisions

Each decision below records the options considered and the choice.

### 1. URL shape

How the URL identifies the object: by bucket and key (the proof of concept), or by catalogue item and asset ID (the feature-requests#16 proposal).

| Option | Outcome | Reason |
|---|---|---|
| Bucket and key, with an allowlist of buckets | **Chosen** | The endpoint needs no catalogue lookup, so it is fast and has no dependency on the catalogue. It builds the S3 target from the path, so it cannot become an open redirect. The catalogue proxy produces these URLs with one `sub_filter` line. |
| Catalogue item and asset ID | Not chosen | Every request needs a catalogue lookup. `sub_filter` cannot build these URLs, so the proxy would need custom code that parses JSON. The S3 key contains a processing date that is not in the item ID, so the key cannot be derived from the ID either. |
| Both, with the item URL as an alias that redirects to the key URL | Not chosen | Two contracts to keep stable, and two redirects for the alias, for no need that the key URL does not meet |

**Decision: the URL names the bucket and the object key.** The endpoint returns 404 for a bucket that is not on the allowlist. Download tracking (feature-requests#16) should use the same endpoint. The URL does not name the catalogue item. If tracking needs the item, it can derive it from the key later.

### 2. Path prefix

Where the route sits under the platform domain, and whether it carries a version.

| Option | Outcome | Reason |
|---|---|---|
| `/api/data/objects/{bucket}/{key}` | **Chosen** | It names the contract, access to a stored object, not the mechanism. Requester-pays becomes a property of a bucket on the allowlist, so download tracking can use the same route for buckets that are not requester-pays. |
| `/api/data/requester-pays/{bucket}/{key}` (the proof of concept) | Not chosen | The name is wrong for any bucket that is not requester-pays, so download tracking would need a second route |
| `/api/data/v1/objects/{bucket}/{key}` | Not chosen | The catalogue proxy generates these URLs on each response, so few are stored. A version can be added beside this path if a breaking change is ever needed. |
| `/api/catalogue/objects/{bucket}/{key}` | Not chosen | It ties a data-access route to the catalogue's namespace |

**Decision: the route is `/api/data/objects/{bucket}/{key}`, with no version.** Each allowlist entry names the bucket, its region, and whether it is requester-pays. For example, the proxy rewrites `s3://usgs-landsat/<key>` to `https://<platform-domain>/api/data/objects/usgs-landsat/<key>`.

### 3. How a request names the workspace

From the token alone, or also from a header or a query parameter. This depends on what the session cookie and API tokens carry.

| Option | Outcome | Reason |
|---|---|---|
| The token's only workspace, with no header or query parameter | Not chosen | It depends on every token being scoped to one workspace. That is not yet confirmed for the session cookie that a link click sends, or for API tokens. |
| The token's only workspace, plus an optional header | Not chosen | A link click cannot send a header, so it fails if the session cookie lists several workspaces |
| A header, then a query parameter, then the token's only workspace (the proof of concept) | **Chosen** | It works whatever the session cookie and API tokens carry, and the proof of concept tested it with every client |

**Decision: the endpoint takes the first of these that is present:**

1. The `X-EODH-Workspace` header
2. The `workspace` query parameter
3. The token's only workspace, if its `workspaces` claim lists exactly one

If none applies, the endpoint returns 400. The chosen workspace must be in the token's `workspaces` claim, or the endpoint returns 403. A workspace name must match `[\w+=,.@-]{2,64}`, because it becomes the STS role session name.

The query parameter is for clients that cannot send a header: a link click in the UI, and a tile URL given to titiler. GDAL clients (`pyeodh`, the QGIS plugin, titiler's own reads) must use the header and set `GDAL_DISABLE_READDIR_ON_OPEN=EMPTY_DIR`, because GDAL's sidecar probes change the query string.

If the session cookie and API tokens turn out to list one workspace each, steps 1 and 2 add nothing, and a later ADR can remove them.

### 4. HEAD and GET behaviour

What each method returns, including status codes and response headers.

**GET returns a redirect without calling AWS.** The endpoint signs the URL locally and returns `307 Temporary Redirect`, with the presigned S3 URL in `Location` and `Cache-Control: no-store`. The presigned URL carries `x-amz-request-payer=requester` in its signed query string, not as a header, because a client that follows a redirect does not send extra headers. A client's `Range` header goes to S3 unchanged when it follows the redirect.

**HEAD is answered by the endpoint itself.** A presigned URL is valid for one method only, and GDAL reuses the target of its first redirect for its later range GETs. So the endpoint never redirects a HEAD. It calls `HeadObject` as the workspace's session and returns `200` with `Content-Length`, `Content-Type`, `ETag`, `Last-Modified` and `Accept-Ranges: bytes`, or `404` if the key does not exist. CloudTrail attributes the `HeadObject` to the workspace.

**A `403` from `HeadObject` becomes `404`.** The signer role has `s3:GetObject` but not `s3:ListBucket`, and the Hub cannot get `s3:ListBucket` on a bucket it does not own. Without it, S3 returns `403` for a missing key, not `404`. A `403` can also mean that the role's policy is wrong, so the endpoint:

- logs the original S3 status and error code with every mapped `404`
- calls `HeadObject` on a known key at startup, as each signing role, and fails its readiness check if that call does not return `200`

| Option for a missing key on GET | Outcome | Reason |
|---|---|---|
| Do not check: sign and redirect | **Chosen** | GDAL and titiler always send HEAD first, so they get a 404 from the endpoint anyway. A check adds a round trip to us-west-2 (about 150 ms) and a billed request to every GET, on titiler's busiest path. |
| Call `HeadObject` first, and return 404 from the endpoint | Not chosen | It helps only clients that do not send HEAD first (curl, browsers, `requests`), and catalogue hrefs point at objects that exist |

A client that skips HEAD and asks for a missing key gets the error from S3, after the redirect. The signer role has `s3:GetObject` but not `s3:ListBucket`, so S3 returns `403 AccessDenied` for a missing key, not `404`.

| Status | Method | When |
|---|---|---|
| `307` | GET | Success: redirect to the presigned URL |
| `200` | HEAD | Success: object metadata |
| `400` | Both | No workspace could be chosen (see decision 3), or the workspace name is not valid |
| `401` | Both | No token, or the token is not valid |
| `403` | Both | The chosen workspace is not in the token's `workspaces` claim |
| `404` | Both | The bucket is not on the allowlist (checked before the token), or, for HEAD, `HeadObject` returned `403` or `404` (see above) |
| `502` | HEAD | `HeadObject` failed for any other reason, for example a timeout or a `5xx` from S3 |

**CORS.** Responses allow any origin, the methods GET, HEAD and OPTIONS, and the request headers `Authorization`, `X-EODH-Workspace` and `Range`. Browser JavaScript still cannot read the data, because CORS is disabled on `usgs-landsat`, but navigation and downloads work.

### 5. Signed URL lifetime

How long a presigned URL stays valid.

A presigned URL stops working at the earlier of two times: its own expiry (`X-Amz-Expires`), and the expiry of the STS session that signed it.

**The lifetime does not limit file size or download time.** S3 checks the signature only when a request starts. A GET that starts before expiry runs to completion, however long it takes. For scale, the proof of concept downloaded one surface reflectance band (`SR_B4`, 71 MB) in 35 seconds. The lifetime was not chosen from file size. It balances how long a leaked URL works against how often clients go back to the endpoint. The lifetime does limit requests that start later with the same URL:

- **Range reads.** Each range read is a new request. GDAL must get a new URL from the endpoint once its cached one expires (see below).
- **Resumed downloads.** A browser download that is paused for longer than the lifetime cannot resume. The user clicks the link again.
- **Download managers that fetch parallel ranges.** They work while the URL is valid. A download that runs past expiry fails if it starts new range requests after that.

The trade-off:

- **A longer lifetime** means fewer requests to the endpoint, because GDAL reuses its cached redirect until the URL expires. But a leaked URL works for longer, billed to the workspace that asked for it, and titiler's cross-user cache window grows.
- **A shorter lifetime** means GDAL goes back to the endpoint more often. This depends on GDAL re-resolving an expired redirect. The GDAL source suggests that it does, but the proof of concept did not test it.

**Decision: the lifetime is a configuration setting, with a default of 15 minutes.** The endpoint enforces one rule between the settings, and refuses to start if they break it: it refreshes a workspace's STS session while at least one URL lifetime remains, so every URL it signs is valid for its full lifetime. With the defaults, the session lasts 1 hour and is refreshed when less than 20 minutes remain. A lifetime longer than about 40 minutes needs a longer session, and so a higher maximum session duration on the signer role.

Before release, a test must show that GDAL goes back to the endpoint when its cached redirect expires, in a QGIS session longer than the lifetime.

### 6. Anonymous thumbnails

How catalogue thumbnails are served without a login, and which objects that route accepts.

Each Landsat item has two preview images in `usgs-landsat`: `thumbnail` (`*_thumb_small.jpeg`, about 5.5 KB) and `reduced_resolution_browse` (`*_thumb_large.jpeg`, larger, size not yet measured). Catalogue pages show them to users who are not logged in. Earth Search's own `/thumbnail` link works only because Element 84 pay for each view, so the Hub must not depend on it.

| Option | Outcome | Reason |
|---|---|---|
| Both preview images are anonymous, at the same URL as any other object | **Chosen** | Catalogue lists and item pages can show either image without a login. The catalogue proxy rewrites their hrefs like any other asset, so no client builds a special URL. |
| Only the small thumbnail is anonymous | Not chosen | Item pages could not show the larger preview to users who are not logged in |
| A separate route, for example `/api/data/thumbnails/...` | Not chosen | `sub_filter` cannot tell thumbnail hrefs from data hrefs, so the UI would have to build these URLs itself |

**Decision: both preview images are served without a login, and the Hub pays for them.**

- **The URL is the same.** A preview image is at `/api/data/objects/usgs-landsat/<key>`, like any other object.
- **A second ingress turns off the login check for preview images only.** Its path regex matches keys that end `_thumb_small.jpeg` or `_thumb_large.jpeg` in an allowlisted bucket. It has its own per-IP rate limit, so that nobody can run up the cost.
- **The endpoint signs these with a separate thumbnail role.** That role's IAM policy allows `s3:GetObject` only on `usgs-landsat/*_thumb_small.jpeg` and `usgs-landsat/*_thumb_large.jpeg`. Even if the endpoint has a bug, the anonymous route cannot sign a read of data. CloudTrail shows thumbnail costs under that role, apart from every workspace's reads, and no workspace is charged. A separate role also avoids a fixed session name that could collide with a workspace name.
- **A token is ignored for preview images.** A logged-in user's preview reads are also the Hub's cost, so the same image costs the same for every user.
- **The redirect may be cached.** The 307 carries `Cache-Control: max-age=300`, so a browser reuses the presigned URL while it is valid and does not ask the endpoint again for every page view. `max-age` must stay shorter than the URL lifetime.

Before release, measure the size of `*_thumb_large.jpeg` and set the rate limit from the expected cost.

## Consequences

**What this makes possible.** The catalogue proxy, the UI, titiler, `pyeodh`, the QGIS plugin, notebooks and ADES stage-in can all be built against one URL shape now. Adding another requester-pays bucket, or a bucket for download tracking, is an allowlist entry and a `sub_filter` line. Catalogue pages keep their thumbnails for users who are not logged in.

**What it costs.**

- **Anonymous reads of Landsat data stop.** Every data read needs a token. The user documentation must say so.
- **The `s3://` hrefs disappear.** The proxy replaces them for every user, including users who could read the bucket with their own AWS credentials.
- **There are three ways to name the workspace.** A client can be charged to a workspace it did not intend, if it sends a header it forgot about. The endpoint logs the chosen workspace on every request.
- **The Hub pays for preview images.** That cost is not recharged to anyone.

**What it does not do.**

- **It does not stop titiler from serving one user's cached redirect to another.** Titiler must check the user's access itself before every read. This is a separate change to titiler.
- **It does not measure bytes.** The endpoint sees about two requests per file. CloudTrail data events measure the bytes, and a separate billing feed turns them into charges.

## Acceptance conditions

The ADR can move to Accepted when:

1. A test shows that GDAL goes back to the endpoint when its cached redirect expires (decision 5).
2. The size of `*_thumb_large.jpeg` is measured, and the thumbnail rate limit is set from it (decision 6).
3. The owner of feature-requests#16 agrees to use this route for download tracking (decision 1).

## Where the detail lives

- [Feasibility: Landsat from Earth Search and requester-pays access](landsat-requester-pays-feasibility.md): the problem, costs and proof-of-concept results
- [EO-DataHub/requester-pays-poc](https://github.com/EO-DataHub/requester-pays-poc) (private): the proof-of-concept code and `RESULTS.md`
- [eodhp-sparkgeo-pm#259](https://github.com/EO-DataHub/eodhp-sparkgeo-pm/issues/259): the findings and the work plan
- [feature-requests#16](https://github.com/EO-DataHub/feature-requests/issues/16): download tracking
