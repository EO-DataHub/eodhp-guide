# Authentication and Authorization

The EO DataHub platform has different ways to authenticate for different use cases. This guide will describe the methodology for each.

## Hub Users

These authentication methods are applicable either when:

- A user is interacting with the EO DataHub directly
- A user's own code is interacting with the EO DataHub on the user's behalf

### Session Authentication

When visiting the EO DataHub in a web browser session authentication will be used. The session can be started by using the "Sign In" link in the web presence. This will start a cookie based session that will be shared between all visits to \*.eodatahub.org.uk. The session will be terminated after 30 minutes of inactivity.

### API Tokens

To authenticate to the EO DataHub APIs, users should use an API token. API tokens can be managed using the interface in the Workspaces section of the web presence.

1. Log into EO DataHub
2. Visit _Workspaces_ page from nav bar
3. Under applications select DataHub API
4. Create a new API Token

API tokens act as offline access tokens. They are valid until deleted.

To authenticate with an API token, include an `Authorization: Bearer <api_token>` header with your request.

You can request as many API tokens as you wish.

### Accessing Workspace

Access to workspace contents can be managed both via a browser with a session cookie and through an API token as described in the API Tokens section. Note that the session cookie for workspaces is different from the cookie used for platform authentication.

Preflighted API requests are also supported.
Unauthenticated requests will be redirected to Keycloak to sign in. Users may only access files in their own workspace or workspace groups that they are members of.

An example request for accessing a workspace file is as follows:

```http
GET /files/<bucket_name>/<path_to_file> HTTP/1.1
Host: <workspace_name>.eodatahub-workspaces.org.uk
Authorization: Bearer <api_token>
```

#### S3 Bucket

Workspace object stores, including saved catalogs and workflow outputs, are accessible at
`https://<workspace_name>.eodatahub-workspaces.org.uk/files/<bucket_name>/<path_to_file>`.
The bucket name and path to workflow outputs may be obtained from ADES outputs.

#### Block Storage

Workspace block stores relevant to AppHub are accessible at
`https://<workspace_name>.eodatahub-workspaces.org.uk/files/workspaces/<path_to_file>`.

## App Developers

EO DataHub apps are tools which are not part of the EO DataHub itself, but which call EO DataHub APIs on behalf of hub users other than the application developer.

OpenID Connect clients can be set up in Keycloak by EO DataHub developers on behalf of application developers (one per app). Please notify the developers of the OIDC code flows you wish to utilise, as these are enabled on an individual basis.

The options in Keycloak are (OIDC terminology in brackets):

- Standard flow (Authorization Code Flow)
- Direct access grant (Resource Owner Password Credentials Grant) [deprecated]
- Implicit flow (Implicit Flow) [deprecated]
- Service account flow (Client Credentials Grant)

> **_Note_**
>
> _The Resource Owner Password Credentials Grant and the Implicit Flow are both deprecated and should not normally be used. Most applications should use the Authorization Code flow._

### Frontend Applications

You will require a public Keycloak client set up for your application by an EO DataHub developer. You will be provided with a `client id` to use to authenticate with the platform.

### Backend Applications

You will require a confidential Keycloak client set up for your application by an EO DataHub developer. You will be provided with a _client id_ and a _client secret_ to use to authenticate with the platform.
