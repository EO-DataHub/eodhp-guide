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

Returns a list of files from one or both stores. Without a `store` parameter, files from both stores are returned. Each item includes `storeType`, `fileName`, `size`, `lastModified`, and `etag` (object store only).

### Upload files

```
POST /workspaces/{workspace-id}/files/object    # → object store (S3)
POST /workspaces/{workspace-id}/files/block     # → block store (EFS)
```

Accepts `multipart/form-data` with one or more files in the `files` field. Returns `201` with an array of uploaded file items on success.

Example using curl:

```bash
curl -X POST \
  "https://eodatahub.org.uk/api/workspaces/<workspace>/files/object" \
  -H "Authorization: Bearer $TOKEN" \
  -F "files=@my-data.tif"
```

### Delete a file

```
DELETE /workspaces/{workspace-id}/files/object?file=<filename>
DELETE /workspaces/{workspace-id}/files/block?file=<filename>
```

The `file` parameter is the filename only — not a path. Returns `200` on success.

### File metadata

```
GET /workspaces/{workspace-id}/files/object/metadata?file=<filename>
GET /workspaces/{workspace-id}/files/block/metadata?file=<filename>
```

Returns size, last-modified timestamp, and etag (object store) for a single file.

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
- **EFS listing** — the block store list endpoint returns all files in a single response with no pagination. For workspaces with a large number of EFS files this may be slow.
