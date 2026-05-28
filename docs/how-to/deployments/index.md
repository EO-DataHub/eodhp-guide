---
title: Deployments
doc_status: ok
last_reviewed: 2026-05-21
reviewed_by: geodowd
review_notes: "Index for deployments how-to guides"
---

# Deployments

Task-focused guides for deploying and maintaining the platform infrastructure and Kubernetes clusters.

**Understanding:** [Kargo and Argo CD — design on EO Data Hub](../../explanation/development-and-governance/kargo-argoc-integration.md) · [GitOps branching and Kargo promotion](../../explanation/development-and-governance/gitops-branching-and-kargo-promotion.md)

## Cluster setup

- [Platform deployment](platform-deployment.md) — step-by-step guide for deploying a new cluster from Terraform through Argo CD bootstrapping and Kargo promotion
- [Bootstrap dependencies](bootstrap-dependencies.md) — sync-wave order and service dependency constraints for cluster initialisation

## Services

- [Deploy eodhp-guide to Kubernetes](deploy-eodhp-guide-to-kubernetes.md) — containerise this MkDocs site and serve it at `/guide` via ArgoCD

## Cluster maintenance

- [Updating cluster Kubernetes version](updating-cluster-kubernetes-version.md)
