---
title: Mixed pages — migration notes
doc_status: unreviewed
---

# Mixed pages

Some existing Markdown mixes Diátaxis modes. During migration **split**, **cross-link**, or add clear in-page headings (**Tutorial**, **How-to**, **Reference**, **Explanation**) if a full split is not worth doing yet.

| Source | Observation | Recommended direction |
|--------|--------------|-----------------------|
| `docs/Development.md` | Describes GitOps model (why) plus points to operational guides (what to do). | Explanation: branching and promotion narrative. How-to: Kargo developer guide link only. |
| `docs/iam/Auth.md` | Mixed concepts (sessions, workspaces URL shape) vs UI procedure (mint API token). | **Implemented in docs2:** [Explanation → Authentication and authorization](iam/authentication-and-authorization.md) + [How-to → Create a Data Hub API token](../how-to/identity-access/create-datahub-api-token.md). Optionally add IAM reference extracts later. |
| `docs/iam/AWS Federated Users.md` | Procedural federation wiring plus sample policies. | [How-to → AWS federated users](../how-to/identity-access/aws-federated-users-oidc.md); link scope context from [OIDC scope design](iam/oidc-scope-design.md). |
| `docs/services/*.md` | Some service pages drift into architectural essays. | Top of page: factual reference; relocate long rationale to explanation (architecture/design-and-decisions) with back-links. |

Revisit after moves: search for long **Note**/`!!!` sections that explain rather than prescribe—those belong in Explanation.
