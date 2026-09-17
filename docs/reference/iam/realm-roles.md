---
title: EODH realm roles
doc_status: needs-verification
tags:
  - keycloak
  - identity
last_reviewed:
reviewed_by:
review_notes:
---
# EODH realm roles

This page lists the platform roles defined or consumed by the `eodhp` Keycloak realm. A realm role is not the same as a Keycloak client role, an OIDC scope, or project membership.

## Platform roles

| Role | Purpose and known use |
|---|---|
| `admin` | Platform administrator role. Used for administrative access to services including Grafana and Kubecost, and by platform authorization policies. It maps to the Grafana **Admin** role. |
| `hub_admin` | Hub administrator role. Used for CMS administration and elevated permissions in some Hub APIs, including account administration. It is not an ordered tier of `admin`. |
| `grafana-viewer` | Read-only Grafana dashboard access. It maps to the Grafana **Viewer** role. When a user also has `admin`, the Grafana Admin mapping takes precedence. |
| `hub_user` | Baseline EO Data Hub user role. It is consumed by platform services for ordinary user access, including workflow-related permissions. |
| `kubecost_operator` | Role accepted by the Kubecost ingress access gate. The configured name uses an underscore. The role's permissions inside Kubecost depend on the deployed configuration. |
| `default-roles-eodhp` | Keycloak's default-role composite for the realm. It assigns the realm's default roles to new users; it is not normally an operator privilege to grant directly. |

The list reflects the roles currently found in the deployment configuration and service policies. The live realm and environment overlays may differ. Update this page when a role or its enforcement changes.

## Assigning roles

An authorized Keycloak administrator can assign platform realm roles from a user's **Role mapping** page. See [Elevate a user](../../how-to/identity-access/elevate-user.md). Use `grafana-viewer` instead of `admin` when dashboard viewing is sufficient. Viewer access does not necessarily restrict the underlying data exposed by dashboards.

## Keycloak administration roles

The `realm-management` client contains Keycloak client roles for administering the `eodhp` realm. These are separate from the platform realm roles above. A platform `admin` or `hub_admin` role does not, by itself, grant access to the Keycloak administration console.

See [Access the eodhp realm console](../../how-to/identity-access/access-eodhp-realm-console.md) for delegated personal-account access.

**Configuration:** [Keycloak deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment/tree/main/apps/keycloak) · [Grafana deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment/tree/main/apps/grafana) · [Kubecost ingress](https://github.com/EO-DataHub/eodhp-argocd-deployment/tree/main/apps/kubecost)
