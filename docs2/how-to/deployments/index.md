---
title: How-to — Deployments
doc_status: unreviewed
---

# Deployments

Infrastructure rollout and prerequisites for environments.

**Primary guide:** [Platform deployment](platform-deployment.md) (Terraform/AWS, EKS kubectl context, ArgoCD bootstrap, Keycloak OAuth, Kargo promotion, SES production access)—adapt workspace/overlay examples for non-prod environments.

**Related lookup:** Core Git repos for cloud and deployment automation are listed under [Deployment repositories](../../reference/deployment-repositories.md).

When migrating from `docs/`:

| Source (current location) | Target (place under this folder after move) |
|---------------------------|---------------------------------------------|
| `docs/Platform Deployment.md` | [platform-deployment.md](platform-deployment.md) (migrated) |
| `docs/operations/bootstrap-dependencies.md` | e.g. `bootstrap-dependencies.md` here |

Operational notes for **applications and GitOps** live under [Kubernetes and GitOps](../kubernetes-gitops/index.md).
