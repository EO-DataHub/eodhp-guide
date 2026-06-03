---
title: Microsoft Entra ID IDP Integration
doc_status: unreviewed
tags:
  - keycloak
  - identity
last_reviewed:
reviewed_by:
review_notes:
---

# Microsoft Entra ID IDP Integration

Explains how Microsoft Entra ID (Azure AD) is configured as an identity provider in Keycloak, the multi-tenant consent model, and what an external organisation's IT admin needs to do before their users can sign in.

---

## How it works

EODH uses Keycloak as its identity broker. Microsoft is configured as an additional IDP alongside GitHub and Google, enabling users to sign in with an Entra ID account from **any** Microsoft organisation — not just a single pre-approved tenant.

The integration uses a **multi-tenant** Azure App Registration, meaning users from any Azure AD directory (including personal Microsoft accounts) can authenticate without EODH needing a separate registration per organisation. However, first-time sign-in from a new tenant requires a one-time admin consent step from that organisation's Entra ID administrator.

---

## Azure App Registration

The Azure side of the integration is a single multi-tenant app registered in the STFC Azure Entra ID tenant:

| Field | Value |
|-------|-------|
| App name | `eodh-multitenant-idp` |
| Supported account types | Accounts in any organizational directory (Any Azure AD directory — Multitenant) |
| API permissions | Microsoft Graph → Delegated: `openid`, `profile`, `email` |
| Redirect URIs | See table below |

**Redirect URIs per environment:**

| Environment | Redirect URI |
|-------------|-------------|
| Test | `https://test.eodatahub.org.uk/keycloak/realms/eodhp/broker/microsoft/endpoint` |
| Staging | `https://staging.eodatahub.org.uk/keycloak/realms/eodhp/broker/microsoft/endpoint` |
| Production | `https://eodatahub.org.uk/keycloak/realms/eodhp/broker/microsoft/endpoint` |

The client ID and client secret are held by the platform team. The client secret is set to expire after 180 days and must be rotated before expiry — see [Client secret expiry](#client-secret-expiry) below.

> **Note:** The `appId` in the Azure manifest export is the authoritative client ID. Always verify from the manifest export if authentication fails — a client ID shared informally may differ.

---

## Keycloak configuration

The IDP is configured in Keycloak as an **OpenID Connect v1.0** identity provider with the following settings:

| Field | Value |
|-------|-------|
| Alias | `microsoft` |
| Display name | `Microsoft` |
| Authorization URL | `https://login.microsoftonline.com/common/oauth2/v2.0/authorize` |
| Token URL | `https://login.microsoftonline.com/common/oauth2/v2.0/token` |
| Logout URL | `https://login.microsoftonline.com/common/oauth2/v2.0/logout` |
| User Info URL | `https://graph.microsoft.com/oidc/userinfo` |
| Issuer | *(leave blank — important)* |
| Client authentication | Client secret sent as basic auth |
| Default Scopes | `openid profile email` |

**The alias must be `microsoft`** — the EODH Keycloak theme (`keycloakify-starter`) uses this alias to render the Microsoft button logo. Any other value (e.g. `azure-oidc`) results in plain text only. The alias cannot be changed after creation; the IDP must be deleted and re-created to change it.

---

## IaC state

The GitHub, Google, and Microsoft IDPs are configured manually via the Keycloak UI. A skeleton exists in `eodhp-argocd-deployment/apps/keycloak/base/realms.yaml` but does not carry the full IDP configuration or secrets. The full Keycloak YAML for the `microsoft` IDP (with secrets redacted) is available from the platform team as a reference for future IaC codification.

---

## Admin consent

Because the app is multi-tenant, first use from a new organisation's Azure AD tenant requires consent from that tenant's Entra ID administrator. There are two scenarios depending on the tenant's policies:

### Scenario A — "Approval required"

The user can initiate a consent request themselves:

1. User attempts to sign in and sees an **Approval required** prompt.
2. User submits the request.
3. The organisation's **Entra ID Global Administrator** navigates to:
   `https://entra.microsoft.com` → **Entra ID** → **Enterprise apps** → **Admin consent requests**
4. Admin finds `eodh-multitenant-idp` in the list.
5. Admin clicks the app → **Review permissions and consent** → **Accept**.

After approval, all users in that tenant can sign in without further intervention.

### Scenario B — "Needs admin approval"

The user cannot initiate a request. The admin must pre-approve by visiting a consent URL directly:

```
https://login.microsoftonline.com/{tenant-id}/v2.0/adminconsent?
  client_id=370f5208-066b-407c-ab51-293387b3eda6
  &redirect_uri=https://eodatahub.org.uk/keycloak/realms/eodhp/broker/microsoft/endpoint
  &scope=370f5208-066b-407c-ab51-293387b3eda6/.default
```

Replace `{tenant-id}` with the **Tenant ID** of the external organisation's Azure AD directory (visible in their Azure Portal → Entra ID → Overview).

The admin logs in with a Global Administrator account and clicks **Accept**. Keycloak may show an error page after the redirect — this is safe to ignore. Once the admin has accepted, all users in that tenant can sign in.

---

## Known limitations

### Admin consent is required per tenant

Unlike GitHub or Google login (which work immediately for any user), Microsoft Entra ID sign-in requires a one-time IT admin action from each external organisation. This is by design for multi-tenant applications — the organisation's admin grants permission for their users to authenticate with an external application.

Allow lead time for IT teams to carry out the approval step when onboarding new organisations.

**Fallback:** Users with a personal Microsoft account (`@outlook.com`, `@hotmail.com`, etc.) can sign in without any tenant-level approval.

### Client secret expiry

The client secret has a 180-day lifetime. When it expires, all Microsoft logins will fail. The platform team must rotate the secret in Azure and update the Keycloak IDP configuration before expiry.

### Keycloak upgrades may affect login page appearance

Keycloak upgrades can affect whether the custom EODH login theme is applied. If the login page appearance changes after an upgrade, verify that the `keycloakify-starter` theme is still active on the realm.

### Username mapping on first sign-up

On first sign-up via Microsoft, Keycloak populates the **Username** field with an email-like string. A custom mapper to suggest a more suitable username has not been implemented.
