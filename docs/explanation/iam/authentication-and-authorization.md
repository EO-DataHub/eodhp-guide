---
title: Authentication and authorization
doc_status: ok
tags:
  - identity
  - oidc
last_reviewed:
reviewed_by:
review_notes: "From docs/iam/Auth.md (conceptual); API token UI steps split to identity-access/how-to."
---

# Authentication and authorization

The EO Data Hub platform has different ways to authenticate for different use cases. This guide summarises each model at a conceptual level. Client and scope inventories live in **[Platform IAM — OIDC clients and scopes](oidc-scope-design.md)**.

## Hub users

These authentication methods are applicable either when:

- A user is interacting with the EO Data Hub directly
- A user's own code is interacting with the EO Data Hub on the user's behalf

### Session authentication

When visiting the EO Data Hub in a web browser, session authentication will be used. The session can be started by using the "Sign In" link in the web presence. This starts a cookie-based session that is shared between all visits to \*.eodatahub.org.uk. The session ends after **30 minutes of inactivity**.

### API tokens

To authenticate to the EO Data Hub APIs, hub users typically use API tokens minted from the workspace UI.

For UI steps (Workspaces → DataHub API → create token), see **[Create a Data Hub API token](../../how-to/identity-access/create-datahub-api-token.md)**.

Include an `Authorization: Bearer <api_token>` header on API requests.

### Accessing workspaces

Access to workspace contents can use both a browser session cookie and an API token. The session cookie for workspaces is **distinct** from the cookie used for hub authentication.

Preflighted API requests are supported. Unauthenticated requests are redirected to Keycloak. Users may only access files in their own workspace or in workspace groups where they are members.

Example request shape:

```http
GET /files/<bucket_name>/<path_to_file> HTTP/1.1
Host: <workspace_name>.eodatahub-workspaces.org.uk
Authorization: Bearer <api_token>
```

#### S3 bucket

Workspace object stores, including saved catalogs and workflow outputs, are reachable at:

`https://<workspace_name>.eodatahub-workspaces.org.uk/files/<bucket_name>/<path_to_file>`

The bucket name and path to workflow outputs may come from ADES outputs.

#### Block storage

Workspace block stores relevant to AppHub are reachable at:

`https://<workspace_name>.eodatahub-workspaces.org.uk/files/workspaces/<path_to_file>`.

## App developers

EO Data Hub apps call EO Data Hub APIs **on behalf of hub users**, not solely for the registering developer.

OpenID Connect clients are provisioned in Keycloak by EO Data Hub operators (typically one client per application). Declare which OIDC flows you need; flows are enabled per client.

Keycloak terminology vs OIDC (parentheses):

- Standard flow (Authorization Code Flow)
- Direct access grant (Resource Owner Password Credentials Grant) [deprecated]
- Implicit flow [deprecated]
- Service account flow (Client Credentials Grant)

> **_Note_**
>
> _The Resource Owner Password Credentials Grant and the Implicit Flow are deprecated and should not normally be used. Most applications should use the Authorization Code flow._

### Frontend applications

Requires a **public** Keycloak client created by EO Data Hub developers. You receive a `client id` for platform authentication.

### Backend applications

Requires a **confidential** Keycloak client. You receive a `client id` and `client secret`.

For detailed client capabilities and scopes, see [Platform IAM — OIDC clients and scopes](oidc-scope-design.md).

For AWS federation and principal-tag patterns, operators may consult [AWS federated users (OIDC and IAM)](../../how-to/identity-access/aws-federated-users-oidc.md).
