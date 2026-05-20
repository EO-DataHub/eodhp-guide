---
title: How-to — Kubernetes and GitOps
doc_status: unreviewed
---

# Kubernetes and GitOps

Cluster operations tied to Kubernetes, Argo CD, Kargo, and similar tooling.

Migrate from `docs/operations/`:

| Source | Suggested filename here |
|--------|-------------------------|
| `operations/argocd/web-ui.md` | `argocd-web-ui.md` |
| `operations/argocd/restart-hub-service.md` | `argocd-restart-hub-service.md` |
| `operations/kargo/developer-guide.md` | `kargo-developer-guide.md` |
| `operations/maintenance/updating-cluster-kubernetes-version.md` | `updating-cluster-kubernetes-version.md` |
| `operations/maintenance/rotating-linkerd-trust-anchor.md` | `rotating-linkerd-trust-anchor.md` |

Migrated from `docs/Development.md` (tasks):

| Guide | Notes |
|-------|-------|
| [Deploy via Argo CD dev branch](deploy-via-argocd-dev-branch.md) | Dev cluster workflow |
| [Debug EKS nodes (SSH via Instance Connect)](debug-eks-node-ssh-instance-connect.md) | EC2 Instance Connect against private subnets |

Link to narrative context (GitOps branch model, warehouses) from [GitOps branching and Kargo promotion](../../explanation/development-and-governance/gitops-branching-and-kargo-promotion.md) ([section index](../../explanation/development-and-governance/index.md)).
