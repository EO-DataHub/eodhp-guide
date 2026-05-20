---
title: How-to — Identity and access
doc_status: unreviewed
---

# Identity and access

Procedures for access changes, federation setup, Keycloak bootstrap, and similar IAM **tasks**.

Migrate from `docs/operations/` and `docs/iam/` — keep **conceptual** IAM under [Explanation → IAM](../../explanation/iam/index.md).

## Identity and access guides

| Guide | Topic |
|-------|--------|
| [Keycloak initial admin access](keycloak-initial-admin-access.md) | Bootstrap access from cluster secret; rotate off default admin |
| [Elevate user](elevate-user.md) | Assign `admin` / `hub_admin` Keycloak realm roles |
| [Onboard an application developer](onboard-app-dev.md) | OIDC clients and Keycloak CSP for third-party apps |
| [Create a Data Hub API token](create-datahub-api-token.md) | Hub user API token (UI) |
| [AWS federated users (OIDC and IAM)](aws-federated-users-oidc.md) | AWS OIDC provider, trust policies, principal tags |

| Legacy source | Notes |
|----------------|-------|
| `iam/AWS Federated Users.md` | [aws-federated-users-oidc.md](aws-federated-users-oidc.md) (migrated) |
| `iam/Auth.md` (UI steps only) | [create-datahub-api-token.md](create-datahub-api-token.md); narrative → [Authentication and authorization](../../explanation/iam/authentication-and-authorization.md) |
| `operations/elevate-user.md` | [elevate-user.md](elevate-user.md) (migrated) |
| `operations/onboard-app-dev.md` | [onboard-app-dev.md](onboard-app-dev.md) (migrated; lives here, not under Notebooks) |
| `operations/keycloak/initial-admin-access.md` | [keycloak-initial-admin-access.md](keycloak-initial-admin-access.md) (migrated) |

Service-specific auth ops can stay beside the owning how-to topic where tightly coupled.
