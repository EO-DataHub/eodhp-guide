# Platform IAM

## OIDC Clients

### eodh

The `eodh` client is responsible for all platform authentication on the eodatahub.org.uk domain.

#### Supported Authentication Flows

- Standard flow
- Direct access grants
- Service account roles

#### Scopes

**Default Scopes**

- openid
- aud
- profile
- email
- groups
- roles

**Optional Scopes**

- workspaces
- workspaces-owned

### eodh-workspaces

The `eodh-workspaces` client is responsible for all workspace authentication, including authorisation on the eodatahub-workspaces.org.uk domain.

#### Supported Authentication Flows

- Standard flow
- Direct access grants
- Service account roles

#### Scopes

**Default Scopes**

- openid
- aud
- profile
- groups
- roles

**Optional Scopes**

- workspaces
- workspace:${workspace}
- user_service:${user_service}

## Scopes

**openid**

This is the default scope returned as a baseline by all OIDC clients.

```
{
  "exp": 1736243921,
  "iat": 1736243621,
  "jti": "f2b9b23a-b4d0-4883-a37c-10e207c0122d",
  "iss": "https://eodatahub.org.uk/keycloak/realms/eodhp",
  "typ": "Bearer",
  "azp": "eodh",
  "sid": "2be73a8c-37cb-4926-a482-43d9cfe07520",
  "scope": ""
}
```

**profile**

This scope provides user info and should normally be provided as default by most OIDC clients.

```
{
  ...
  "name": "Joe Bloggs",
  "preferred_username": "jbloggs",
  "given_name": "Joe",
  "family_name": "Bloggs",
}
```

**email**

This scope provides user email, and is required for `oauth2-proxy` OIDC clients.

```
{
  ...
  "email": "joebloggs@eodatahub.org.uk",
}
```

**aud**

The audience of clients to which the access token can be used.

```
{
  ...
  "aud": ["client1", "client2"]
}
```

**groups**

This scope provides the Keycloak groups that a user is a member of.

```
{
  ...
  "groups": ["group1", "group2"]
}
```

**roles**

This scope provides the Keycloak roles that a group is permitted for.

```
{
  ...
  "roles": {
    "realm_access": {
      "roles": ["role1", "role2"]
    }
  }
}
```

**workspaces**

This scope provides available workspaces available to the user. It is used to grant access to non-chargeable workspace resources for the user.

```
{
  ...
  "workspaces": ["workspace1", "workspace2"]
}
```

**workspaces-owned**

This scope provides authorisation for user owned workspaces, which allows additional admin permissions over in addition to the workspace scope.

```
{
  ...
  "workspaces-owned": ["workspace1", "workspace2"]
}
```

**workspace:${workspace}**

This is a dynamic scope that will designate the active workspace, if authorised. The desired active workspace is passed as part of the token request as a dynamic scope parameter after the ':'. If `workspaces` scope is also requested then the available workspaces are filtered to only include the active workspace.

The scope also contains an additional AWS scope to allow for parameterised AWS policies when using `assumeRoleWithWebIdentity` using principal tags.

```
{
  ...
  "workspace": "workspace1",
  "https://aws.amazon.com/tags": {
    "principal_tags": {
      "workspaces": ["workspace1"]
    }
  }
}
```

**user_services:${user_service}**
This is a dynamic scope that will return the active user_service. There is no additional authorisation required for this. This scope is intended for use only by authorised OIDC clients as machine users.

The scope also contains an additional AWS scope to allow for parameterised AWS policies when using `assumeRoleWithWebIdentity` using principal tags.

```
{
  ...
  "user_service": "user_service1",
  "https://aws.amazon.com/tags": {
    "principal_tags": {
      "user_services": ["user_service1"]
    }
  }
}
```
