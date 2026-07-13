---
title: Deployment repositories
doc_status: ok
last_reviewed: 2026-05-20
reviewed_by: geodowd
review_notes: "Adapted from docs/README.md (Deployment Repositories)"
tags:
  - deployment
  - gitops
---

# Deployment repositories

Core Git repositories used when standing up managed infrastructure and deployments for the EO Data Hub.

Infrastructure rollout procedures are summarised under [Deployments how-to](../how-to/deployments/index.md).

- <https://github.com/EO-DataHub/eodhp-deploy-supporting-infrastructure> — Initialises an AWS instance with any resources to be shared by deployed environments.
- <https://github.com/EO-DataHub/eodhp-deploy-infrastucture> — Manages infrastructure for an individual deployment environment, including initial deployment. It also bootstraps the Kubernetes cluster, including some required resources, drivers and operators.
- <https://github.com/EO-DataHub/eodhp-argocd-deployment> — Manages the deployment environment services and resources using GitOps.

For a fuller inventory of EO Data Hub–related repos, see the [repository catalogue](repositories.md).
