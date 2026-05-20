---
title: Explanation — Development and governance
doc_status: unreviewed
---

# Development and governance

How engineering works on EO Data Hub: GitOps branching model, promotion concepts, workflows at a conceptual level—not copy-pasted shell steps.

Migrate and split carefully:

| Source | Action |
|--------|--------|
| `docs/Development.md` | **Concept:** [GitOps branching and Kargo promotion](gitops-branching-and-kargo-promotion.md). **PR workflow:** [Developer branching and PR workflow](../../contributing/processes/development-branching-and-pr-workflow.md). **Tasks:** [Deploy via dev branch](../../how-to/kubernetes-gitops/deploy-via-argocd-dev-branch.md), [Debug EKS nodes](../../how-to/kubernetes-gitops/debug-eks-node-ssh-instance-connect.md) under Kubernetes and GitOps. |

Link out to procedural detail rather than nesting full runbooks inside this quadrant.
