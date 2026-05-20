---
title: Mixed pages — migration notes
doc_status: unreviewed
---

# Mixed pages

Some existing Markdown mixes Diátaxis modes. During migration **split**, **cross-link**, or add clear in-page headings (**Tutorial**, **How-to**, **Reference**, **Explanation**) if a full split is not worth doing yet.

| Source | Observation | Recommended direction |
|--------|--------------|-----------------------|
| `docs/Development.md` | Describes GitOps model (why) plus points to operational guides (what to do). | Explanation: branching and promotion narrative. How-to: Kargo developer guide link only. |
| `docs/iam/Auth.md` | Often spans architecture and procedures. | Split facts → reference/services or explanation/iam; steps → how-to/identity-access. |
| `docs/iam/AWS Federated Users.md` | Can be procedural (how-to); may include background. | Leading section in how-to; link “Why” paragraphs to explanation/iam or architecture. |
| `docs/services/*.md` | Some service pages drift into architectural essays. | Top of page: factual reference; relocate long rationale to explanation (architecture/design-and-decisions) with back-links. |

Revisit after moves: search for long **Note**/`!!!` sections that explain rather than prescribe—those belong in Explanation.
