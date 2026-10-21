---
title: Design decisions
doc_status: unreviewed
last_reviewed:
reviewed_by:
review_notes: "Updated 2026-10-21 against shipped code."
---

# Design decisions

Decision records for cross-cutting platform design — the "what was decided and why" for features that span multiple repos, kept separately from the narrative background reading elsewhere in [Explanation](../index.md).

Expect pages here to read like a decision log (context, options considered, decision, consequences) rather than a finished explanation of how something already works — that's the main difference from the rest of this section.

## Workspace roles, ownership, visibility & accounting

This covers Members, Publisher, Files, Profile, Platform admin usage, and the Accounting/Billing pages (Credits, Usage, Budget). The Members, Credits and Usage pages are built, and the back end can set a workspace category. Publisher and Files visibility, Platform admin and Budget remain proposals only. The two UX pages describe what is on `develop`, with the proposals listed at the end of each.

- [Workspace management UX](workspace-management-ux.md) — the Profile, Members, Credentials and Provider accounts pages as built, who can do what, and the proposals not yet built (Publisher/Files visibility, platform admin)
- [Accounting & billing UX](accounting-billing-ux.md) — the Credits, Usage and Billing accounts pages as built, and the proposals not yet built (Budget, platform costs, category display)
- [Proposed backend endpoints](pending-backend-endpoints.md) — flat, scannable table of every backend endpoint needed, grouped by page, with which are built
- [Accounting & billing backend ADR](accounting-billing-backend-adr.md) — Architecture Decision Record describing the back end changes, proposed and implemented, and the reasoning behind them

## Credits ledger implementation

Lower-level design work for the credit-based accounting model in `accounting-service`, one level down from the ADR and UX pages above.

- [Credits ledger design decisions](credits-ledger-design-decisions.md) — numbered decisions for the credits, ledger, and budget features, and what each changes in the task list
- [Credits ledger schema](credits-ledger-schema.md) — database schema implementing those decisions, plus the API response conventions new endpoints should follow
- [Credits ledger scoping](credits-ledger-scoping.md) — implementation tasks for the credits, ledger, and account system, sequenced against findings in the current codebase
- [Credit rate derivation](credit-rate-derivation.md) — how each credits-per-unit rate is calculated from AWS eu-west-2 list pricing, and what the rates do not cover
- [Dev tooling notes](dev-tooling-notes.md) — working notes for the `accounting-service` dev environment, kept separate from the decision records above

## Data access

- [ADR-002: Bulk download of item assets in the browser](bulk-download-adr.md) — why users download many files of one item from the browser, with scripts for large files, instead of through a backend zip service
