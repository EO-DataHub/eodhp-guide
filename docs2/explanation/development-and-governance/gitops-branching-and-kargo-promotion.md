---
title: GitOps branching and Kargo promotion
doc_status: ok
last_reviewed: 2026-05-20
reviewed_by: gedowd
review_notes: "Conceptual sections from docs/Development.md"
---

# GitOps branching and Kargo promotion

How application configuration flows from `main` into environments via Argo CD and Kargo—not step-by-step runbooks.

For day-to-day branching habits and PRs on `eodhp-argocd-deployment`, see [Developer branching and PR workflow](../../contributing/processes/development-branching-and-pr-workflow.md). For tasks (dev cluster deploy, EKS node SSH), see [Kubernetes and GitOps how-to](../../how-to/kubernetes-gitops/index.md).

## Argo CD deployment

The configuration of each environment is controlled by the `eodhp-argocd-deployment` repo. The deployment is managed by Argo CD in a GitOps methodology, with [Kargo](https://kargo.io) handling promotion of changes between environments.

## Branch and promotion model

Developers commit changes to the `main` branch. Kargo detects changes and manages promotion through environments:

1. **Commit to `main`** — developers merge feature branches into `main` via PR
> [!NOTE]
> The default merge method is set to `Squash and merge` to mitigate possible conflicts of two PRs updating the same manifests. Kargo works best with linear history so "Rebase and merge" would work too.
2. **Auto-promote to test** — Kargo detects the change and automatically promotes to the test environment
3. **Manual promotion to staging** — promote via the [Kargo UI](https://kargo.eodatahub.org.uk) after verifying in test
4. **Manual promotion to prod** — promote via the Kargo UI after verifying in staging

Each environment reads from a dedicated `kargo/<app>/<env>` branch (e.g. `kargo/accounting-service/test`). These branches are managed entirely by Kargo — developers should not commit to them directly.

**Important:** Container image versions and Helm chart versions are managed by Kargo warehouses, which automatically detect new tags and versions from their respective registries. Manually editing image tags or chart versions in the deployment repo will not deploy those versions — they will be overwritten on the next Kargo promotion. To deploy a new image or chart version, push the artifact to its registry and let Kargo pick it up (see the developer guide for details).

For how Kargo and Argo CD fit together end-to-end, see [Kargo and Argo CD — design on EO Data Hub](kargo-argoc-integration.md). For warehouses, pipelines, overlays, and day-to-day developer tasks, see the [Kargo developer guide](../../how-to/kubernetes-gitops/kargo-developer-guide.md).

## Applications and packages

All other code repositories, including the Terraform infrastructure deployment repos, follow a more conventional GithubFlow as follows:

- feature branch → `main`: PR required (any peer)
- release from `main` using GitHub Actions (must include git tagging of release commit)

The updated release can then be included in the Argo CD deployment repo following the methodology above.

## Related how-to guides

- [Deploy via Argo CD dev branch](../../how-to/kubernetes-gitops/deploy-via-argocd-dev-branch.md)
- [Debug EKS nodes (SSH via Instance Connect)](../../how-to/kubernetes-gitops/debug-eks-node-ssh-instance-connect.md)
