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
