---
title: Design decisions
doc_status: unreviewed
last_reviewed:
reviewed_by:
review_notes: "New section, no content migrated yet"
---

# Design decisions

Decision records for cross-cutting platform design — the "what was decided and why" for features that span multiple repos, kept separately from the narrative background reading elsewhere in [Explanation](../index.md).

Expect pages here to read like a decision log (context, options considered, decision, consequences) rather than a finished explanation of how something already works — that's the main difference from the rest of this section.

## Workspace roles, ownership, visibility & accounting

This covers Members, Publisher, Files, Profile, Platform admin usage, and the Accounting/Billing pages (Credits, Usage, Budget).

- [Workspace management UX](workspace-management-ux.md) — what's changing in Members, Publisher/Files visibility, Profile, and platform-admin controls, and why, with open questions for client review
- [Accounting & billing UX](accounting-billing-ux.md) — the proposed credit-based Credits/Usage/Budget pages and the workspace category setting, with open questions for client review
- [Proposed backend endpoints](pending-backend-endpoints.md) — flat, scannable table of every backend endpoint needed, grouped by page
- [Accounting & billing backend ADR](accounting-billing-backend-adr.md) —  Architecture Decision Record describing the proposed back end changes and the reasoning behind them

## Credits ledger implementation

Lower-level design work for the credit-based accounting model in `accounting-service`, one level down from the ADR and UX pages above.

- [Credits ledger design decisions](credits-ledger-design-decisions.md) — numbered decisions for the credits, ledger, and budget features, and what each changes in the task list
- [Credits ledger schema](credits-ledger-schema.md) — database schema implementing those decisions, plus the API response conventions new endpoints should follow
- [Credits ledger scoping](credits-ledger-scoping.md) — implementation tasks for the credits, ledger, and account system, sequenced against findings in the current codebase
- [Dev tooling notes](dev-tooling-notes.md) — working notes for the `accounting-service` dev environment, kept separate from the decision records above
