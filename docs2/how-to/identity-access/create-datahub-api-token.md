---
title: Create a Data Hub API token
doc_status: ok
last_reviewed:
reviewed_by:
review_notes: "Procedural excerpt from docs/iam/Auth.md"
---

# Create a Data Hub API token

API tokens are issued from the web UI under Workspaces → applications.

## Steps

1. Log into the EO Data Hub
2. Open **Workspaces** from the navigation bar
3. Under applications, choose **DataHub API**
4. Create a new API token

Tokens behave as **offline-style** access secrets: they stay valid until you delete them.

## Using the token

Send `Authorization: Bearer <api_token>` with each HTTP request.

You may create multiple tokens.

**Context:** [Authentication and authorization](../../explanation/iam/authentication-and-authorization.md)
