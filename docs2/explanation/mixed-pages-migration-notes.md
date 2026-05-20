---
title: Mixed pages — migration notes
doc_status: unreviewed
---

# Mixed pages

Some existing Markdown mixes Diátaxis modes. During migration **split**, **cross-link**, or add clear in-page headings (**Tutorial**, **How-to**, **Reference**, **Explanation**) if a full split is not worth doing yet.

| Source | Observation | Recommended direction |
|--------|--------------|-----------------------|
| `docs/Development.md` | Describes GitOps model (why) plus points to operational guides (what to do). | Explanation: [Git branching and promotion](development-and-governance/gitops-branching-and-kargo-promotion.md); How-to: [Kargo developer guide](../how-to/kubernetes-gitops/kargo-developer-guide.md), design in [Kargo and Argo CD](development-and-governance/kargo-argoc-integration.md). |
| `docs/operations/kargo/developer-guide.md` | Explainer (concepts/topology/task breakdown) mixed with onboarding recipes. | **Split in docs2:** [Explanation → Kargo and Argo CD](development-and-governance/kargo-argoc-integration.md) + [How-to → Kargo developer guide](../how-to/kubernetes-gitops/kargo-developer-guide.md). |
| `docs/iam/Auth.md` | Mixed concepts (sessions, workspaces URL shape) vs UI procedure (mint API token). | **Implemented in docs2:** [Explanation → Authentication and authorization](iam/authentication-and-authorization.md) + [How-to → Create a Data Hub API token](../how-to/identity-access/create-datahub-api-token.md). Optionally add IAM reference extracts later. |
| `docs/iam/AWS Federated Users.md` | Procedural federation wiring plus sample policies. | [How-to → AWS federated users](../how-to/identity-access/aws-federated-users-oidc.md); link scope context from [OIDC scope design](iam/oidc-scope-design.md). |
| `docs/services/*.md` | Some service pages drift into architectural essays. | Top of page: factual reference; relocate long rationale to explanation (architecture/design-and-decisions) with back-links. |
| `docs/operations/maintenance/updating-cluster-kubernetes-version.md` | **Purpose** block explains upgrade policy; remainder is procedural. | Stay one how-to under [Deployments](../how-to/deployments/updating-cluster-kubernetes-version.md); headings already separate rationale from steps. |
| `docs/operations/maintenance/rotating-linkerd-trust-anchor.md` | Failure-mode paragraph is risk context ahead of rollout steps. | Stay one how-to under [Kubernetes and GitOps](../how-to/kubernetes-gitops/rotating-linkerd-trust-anchor.md). |

Revisit after moves: search for long **Note**/`!!!` sections that explain rather than prescribe—those belong in Explanation.
