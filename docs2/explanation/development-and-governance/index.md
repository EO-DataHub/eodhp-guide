---
title: Explanation — Development and governance
doc_status: unreviewed
---

# Development and governance

How engineering works on EO Data Hub: GitOps branching model, promotion concepts, workflows at a conceptual level—not copy-pasted shell steps.

## Guides

| Page | Topic |
|------|--------|
| [GitOps branching and Kargo promotion](gitops-branching-and-kargo-promotion.md) | Branches, environments, warehouses at a conceptual level |
| [Kargo and Argo CD — design on EO Data Hub](kargo-argoc-integration.md) | EO Data Hub Kargo topology, warehouses/stages/PromotionTask mechanics |

Migrate and split carefully:

| Source | Action |
|--------|--------|
| `docs/Development.md` | **Concept:** [GitOps branching and Kargo promotion](gitops-branching-and-kargo-promotion.md). **PR workflow:** [Developer branching and PR workflow](../../contributing/processes/development-branching-and-pr-workflow.md). **Tasks:** [Deploy via dev branch](../../how-to/kubernetes-gitops/deploy-via-argocd-dev-branch.md), [Debug EKS nodes](../../how-to/kubernetes-gitops/debug-eks-node-ssh-instance-connect.md) under Kubernetes and GitOps. |
| `docs/operations/kargo/developer-guide.md` | Split: [Kargo design](kargo-argoc-integration.md) + **[Kargo developer guide](../../how-to/kubernetes-gitops/kargo-developer-guide.md)** (tasks). |

Link out to procedural detail rather than nesting full runbooks inside this quadrant.
