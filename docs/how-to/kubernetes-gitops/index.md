---
title: Kubernetes & GitOps
doc_status: ok
last_reviewed: 2026-05-21
reviewed_by: geodowd
review_notes: "Index for kubernetes-gitops how-to guides"
---

# Kubernetes & GitOps

Task-focused guides for deploying and operating the platform via Argo CD, Kargo, and kubectl.

**Understanding:** [GitOps branching and Kargo promotion](../../explanation/development-and-governance/gitops-branching-and-kargo-promotion.md) · [Kargo and Argo CD — design on EO Data Hub](../../explanation/development-and-governance/kargo-argoc-integration.md)

## Argo CD

- [Access the Argo CD web UI](argocd-web-ui.md)
- [Deploy via Argo CD dev branch](deploy-via-argocd-dev-branch.md)
- [Restart a Hub service using Argo CD](argocd-restart-hub-service.md)

## Kargo

- [Kargo developer guide](kargo-developer-guide.md) — deploying images and config changes, adding new apps, Kustomize overlays

## Cluster operations

- [Debug EKS nodes (SSH via EC2 Instance Connect)](debug-eks-node-ssh-instance-connect.md)
- [Rotate the Linkerd trust anchor](rotating-linkerd-trust-anchor.md)
