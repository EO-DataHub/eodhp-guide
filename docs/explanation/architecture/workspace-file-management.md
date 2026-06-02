---
title: Workspace File Management
doc_status: unreviewed
tags:
  - workspaces
  - aws
last_reviewed:
reviewed_by:
review_notes:
---

# Workspace File Management

Explains how users can upload, list, and delete files in their workspace storage without using JupyterHub, covering the file management API and the S3 credentials approach.

**Reference:** [Workspaces service](../../reference/services/workspaces.md) · [Shared storage allocations](../../reference/services/shared-storage-allocations.md)

---

## Storage types

Each workspace has two storage areas:

| Store | Type | Best for |
|-------|------|----------|
| Object store | S3 (`s3://workspaces-eodhp/<workspace>/`) | Large files, COGs, STAC assets, purchased data |
| Block store | EFS (POSIX filesystem) | Notebooks, scripts, small working files; mounted as home directory in JupyterHub |

JupyterHub users can manage files in both stores through the built-in file manager. The file management API and S3 credentials feature are aimed at users who want to upload or manage data **without launching a notebook server**, and at integrations that need programmatic file access.

---

## File management API

The workspace services API exposes endpoints for uploading, listing, deleting, and inspecting files in both stores. All endpoints require a Keycloak Bearer token:

```
Authorization: Bearer <keycloak-token>
```

### List files

```
GET /workspaces/{workspace-id}/files
GET /workspaces/{workspace-id}/files?store=object
GET /workspaces/{workspace-id}/files?store=block
```

Returns a list of files from one or both stores. Without a `store` parameter, files from both stores are returned.

**Response:**
```json
{
  "workspace": "my-workspace",
  "items": [
    { "storeType": "object", "fileName": "my-data.tif", "size": 104857600, "lastModified": "2026-06-01T12:00:00Z", "etag": "abc123" },
    { "storeType": "block",  "fileName": "notebook.ipynb", "size": 4096, "lastModified": "2026-06-01T11:00:00Z" }
  ]
}
```

Note: `etag` is only present for object store items. Block store listings return all files in a single response with no pagination — see [Limitations](#limitations).

### Upload files

```
POST /workspaces/{workspace-id}/files/object    # → object store (S3)
POST /workspaces/{workspace-id}/files/block     # → block store (EFS)
```

Accepts `multipart/form-data` with one or more files in the `files` field. Returns `201` with an array of uploaded file items on success.

**Upload size limit:** Individual files are capped at approximately **5 GB** per request. Requests exceeding this return `413`. For files larger than 5 GB, use the [S3 credentials approach](#s3-credentials) with the AWS CLI, which handles multipart upload automatically.

**Block store timeout:** Block store uploads have a 30-second timeout. Large files may fail rather than just being slow — use S3 credentials for large block store uploads.

Example using curl:

```bash
curl -X POST \
  "https://eodatahub.org.uk/api/workspaces/<workspace>/files/object" \
  -H "Authorization: Bearer $TOKEN" \
  -F "files=@my-data.tif"
```

**Response:**
```json
{
  "workspace": "my-workspace",
  "items": [
    { "storeType": "object", "fileName": "my-data.tif", "size": 104857600 }
  ]
}
```

### Delete a file

```
DELETE /workspaces/{workspace-id}/files/object?file=<filename>
DELETE /workspaces/{workspace-id}/files/block?file=<filename>
```

The `file` parameter is the filename only — not a path. Returns `200` on success, or `409` on partial failure (when the file could not be deleted). In the partial failure case the response body still includes `deleted` and `failed` arrays indicating what succeeded and what did not.

**Response (200 — success):**
```json
{
  "workspace": "my-workspace",
  "deleted": ["my-data.tif"]
}
```

**Response (409 — partial failure):**
```json
{
  "workspace": "my-workspace",
  "deleted": [],
  "failed": [
    { "fileName": "my-data.tif", "error": "failed to delete object" }
  ]
}
```

### File metadata

```
GET /workspaces/{workspace-id}/files/object/metadata?file=<filename>
GET /workspaces/{workspace-id}/files/block/metadata?file=<filename>
```

Returns size, last-modified timestamp, and etag (object store) for a single file.

**Response:**
```json
{
  "workspace": "my-workspace",
  "item": { "storeType": "object", "fileName": "my-data.tif", "size": 104857600, "lastModified": "2026-06-01T12:00:00Z", "etag": "abc123" }
}
```

### File name constraints

All endpoints apply the same validation:

- Maximum 255 bytes
- No path separators — filenames only, not paths (nested directories are not supported)
- No leading dots
- No `..` sequences

---

## S3 credentials

As an alternative to the file management API, users can request temporary AWS credentials to interact with the workspace object store directly using the AWS CLI or SDK.

### Getting credentials

In the **Workspace UI**, navigate to the **S3 credentials** section and click **Request Temporary AWS S3 Credentials**. The UI displays the `accessKeyId`, `secretAccessKey`, and `sessionToken` with copy buttons, and supports copying all three in `.env` format.

Credentials can also be requested via API:

```
POST /api/workspaces/{workspace-id}/me/s3-tokens
```

The response includes `accessKeyId`, `secretAccessKey`, `sessionToken`, and `expiration`.

Credentials are short-lived (session tokens expire). Copy them when generated — they are only displayed once in the UI.

### Using credentials with the AWS CLI

```bash
export AWS_ACCESS_KEY_ID=<accessKeyId>
export AWS_SECRET_ACCESS_KEY=<secretAccessKey>
export AWS_SESSION_TOKEN=<sessionToken>

# List workspace files
aws s3 ls s3://workspaces-eodhp/<workspace>/

# Upload a file
aws s3 cp my-data.tif s3://workspaces-eodhp/<workspace>/my-data.tif
```

---

## Authentication

Both the file management API and S3 credentials endpoint use the user's Keycloak token. The workspace services exchange this token for workspace-scoped AWS credentials via `AssumeRoleWithWebIdentity`, scoping S3 access to the requesting user's workspace.

---

## Limitations

- **No nested directories via the API** — the file management endpoints accept flat filenames only. To organise files in subdirectories, use the S3 credentials approach with the AWS CLI or SDK.
- **5 GB upload cap** — files larger than approximately 5 GB are rejected with `413`. Use the S3 credentials approach with the AWS CLI for larger files (the CLI handles multipart upload automatically). The block store has no explicit size cap but is subject to the 30-second timeout.
- **Block store 30-second timeout** — all block store operations (upload, listing) time out after 30 seconds. Large file uploads or listings of very large directories may fail rather than just being slow; use S3 credentials for large block store uploads.
- **EFS listing** — the block store list endpoint returns all files in a single response with no pagination. For workspaces with a large number of EFS files this may be slow.
