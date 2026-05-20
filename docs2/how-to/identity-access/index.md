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
| [Create a Data Hub API token](create-datahub-api-token.md) | UI steps for token minting (split from Auth) |

| Legacy source | Notes |
|----------------|-------|
| `operations/elevate-user.md` | Place here when migrated |
| `operations/keycloak/initial-admin-access.md` | Place here when migrated |
| `iam/AWS Federated Users.md` | [aws-federated-users-oidc.md](aws-federated-users-oidc.md) (migrated) |
| `iam/Auth.md` (UI steps only) | [create-datahub-api-token.md](create-datahub-api-token.md); narrative → [Authentication and authorization](../../explanation/iam/authentication-and-authorization.md) |

Service-specific auth ops can stay beside the owning how-to topic where tightly coupled.
