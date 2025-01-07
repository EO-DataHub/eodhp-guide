# Platform IAM

## OIDC Clients

### eodh (Formerly oauth2-proxy)

The `eodh` client is responsible for all platform authentication on the eodatahub.org.uk domain.

#### Supported Authentication Flows

- Standard flow
- Direct access grants

#### Scopes

**Default Scopes**

- openid
- profile

**Optional Scopes**

- groups
- roles

### eodh-workspaces (Formerly oauth2-proxy-workspaces)

The `eodh-workspaces` client is responsible for all workspace authentication, including authorisation on the eodatahub-workspaces.org.uk domain.

#### Supported Authentication Flows

- Standard flow
- Direct access grants
- Service account roles

#### Scopes

**Default Scopes**

- openid
- profile

**Optional Scopes**

- groups
- roles
- workspaces
- workspace:${workspace}

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
  "name": "Steven Gillies",
  "preferred_username": "sgillies-tpzuk",
  "given_name": "Steven",
  "family_name": "Gillies",
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
  "roles": ["role1", "role2"]
}
```

**workspaces**
This scope provides available workspaces without setting the active workspace.

```
{
  ...
  "workspaces": {
    "active": null,
    "available": ["workspace1", "workspace2"]
  }
}
```

**workspaces:${workspace}**
This is a dynamic scope that will return an active workspace, if authorised. The available workspaces are included with this as well. The desired active workspace is passed as part of the token request as a dynamic scope parameter after the ':'.

```
{
  ...
  "workspaces": {
    "active": "workspace1",
    "available": ["workspace1", "workspace2"]
  }
}
```
