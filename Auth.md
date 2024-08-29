# Authentication and Authorization

The EO DataHub platform has different ways to authenticate for different use cases. This guide will describe the methodology for each.

## Hub Users

### Session Authentication

When visiting the EO DataHub in a web browser session authentication will be used. The session can be started by using the "Sign In" link in the web presence. This will start a cookie based session that will be shared between all visits to \*.eodatahub.org.uk. The session will be terminated after 30 minutes of inactivity.

### API Tokens

To authenticate to the EO DataHub APIs, users should use an API token. API tokens can be managed using the interface in the Workspaces section of the web presence (not implemented yet).

> **_Note_** > _Until the web interface for managing API tokens is ready, it is necessary to request them from an API. This API is not intended to be available to users in the final EO DataHub, and will be protected from public access once the web UI is available._
>
> _To request an API token, you must have access to a valid OIDC access token for your user. You can request one of these with a direct access grant to Keycloak's ODC endpoint:_
>
>     curl -L 'https://test.eodatahub.org.uk/keycloak/realms/eodhp/protocol/openid-connect/token' \
>         -H 'Content-Type: application/x-www-form-urlencoded' \
>         --d 'client_id=<client_id>' \
>         --d 'client_secret=<client_secret>' \
>         --d 'username=<username>' \
>         --d 'password=<password>' \
>         --d 'grant_type=password'
>
> _With a valid access token, you can request an API token using:_
>
>     curl -L -X POST 'https://test.eodatahub.org.uk/api/tokens' \
>         -H 'Authorization: Bearer <access_token>'
>
> _You should take note of the API token as you will not be able to retrieve it again (it will not appear in any GET requests)._
>
> _The following endpoints are available to manage your tokens:_
>
> - [GET, POST] https://test.eodatahub.org.uk/api/tokens
> - [GET, DELETE] https://test.eodatahub.org.uk/api/tokens/<token_id>

API tokens act as offline access tokens. They are valid until deleted.

To authenticate with an API token, include an `Authorization: Bearer <api_token>` header with your request.

You can request as many API tokens as you wish.

### Accessing Workspace

#### S3 Bucket

#### Block Storage

## App Developers

Keycloak clients can be set up by EO DataHub developers on behalf of application developers (one per app). Please notify the developers of the OIDC code flows you wish to utilise, as these are enabled on an individual basis.

The options in Keycloak are (OIDC terminology in brackets):

- Standard flow (Authorization Code Flow)
- Direct access grant (Resource Owner Password Credentials Grant)
- Implicit flow (Implicit Flow)
- Service account flow (Client Credentials Grant)

### Frontend Applications

You will require a public Keycloak client set up for your application by an EO DataHub developer. You will be provided with a `client id` to use to authenticate with the platform.

### Backend Applications

You will require a confidential Keycloak client set up for your application by an EO DataHub developer. You will be provided with a `client id` and a `client secret` to use to authenticate with the platform.
