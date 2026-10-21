---
title: Accounting & billing UX
doc_status: unreviewed
tags:
  - rc-ui
  - workspaces
last_reviewed:
reviewed_by:
review_notes: "Rewritten 2026-10-21 to describe eodhp-rc-ui develop"
---

# Accounting & billing UX

The old workspace app billed in pounds and showed a flat list of invoices. The Resource Catalogue app (`eodhp-rc-ui`) now uses a shared **credit** currency. A workspace holds a credit balance and spends it on compute at a per-resource rate. This page describes the accounting pages on the `develop` branch: what each page shows, which API it calls, and why it works that way where the code or pull requests say so.

All pages call real endpoints on `accounting-service` and `eodhp-workspace-services`. There are no mocks. The Credits and Usage pages were added in rc-ui #611 and released in #613.

## Navigation

The username menu has an **Accounting / Billing** submenu with two entries. Both open in the `/accounting` sidebar layout.

| Route | Page | Code |
|---|---|---|
| `/accounting/credits` | Credits | `src/pages/CreditsPage/CreditsPage.tsx` |
| `/accounting/usage` | Usage | `src/pages/UsagePage/UsagePage.tsx` |

Both pages show data for the workspace selected in the top-nav workspace switcher. With no workspace selected, they ask the user to select one.

Billing accounts are a separate page under Profile. See [Billing accounts](#billing-accounts).

## Who can see what

`accounting-service` gives every workspace endpoint used here the `MEMBER` minimum tier. A `hub_admin` passes every check. The rates endpoints only need a valid token.

| Data | Endpoint | Minimum access |
|---|---|---|
| Balance | `GET /api/workspaces/:ws/accounting/balance` | Workspace member |
| Ledger (top-ups) | `GET /api/workspaces/:ws/accounting/ledger` | Workspace member |
| Usage events | `GET /api/workspaces/:ws/accounting/usage-data` | Workspace member |
| Rates | `GET /api/accounting/pricing-policy` | Signed in |
| Resource names and units | `GET /api/accounting/skus` | Signed in |

Neither page has write actions. Members, admins and the owner see the same thing.

## Credits

The Credits page has four parts.

**Balance.** The current balance for the workspace, to two decimal places, with an "as of" date. A negative balance gets a red "Negative" badge and a warning that new activity may be blocked until the balance is topped up.

**Credit history.** Top-ups for the workspace, read from the ledger with `type=grant`. Each row shows date, amount and reason. The page shows the five most recent. "View all" opens a paginated list of up to 100. The page sorts grants by date itself and does not rely on the order the API returns.

**Credit rates.** One row per SKU in the current pricing policy, with two columns:

- **Credits per unit**, for example credits per CPU-second.
- **Units per credit**, the inverse. This is easier to read when the per-unit rate is a very small number.

Resource names and units come from `/api/accounting/skus`. The table shows base rates. It does not apply the workspace category multiplier, because the app cannot read which category a workspace is in. The page says this under the table. `eodhp-workspace-services` has `PUT /workspaces/:ws/category` for hub admins, but no endpoint that returns a workspace's category.

**Buying credits.** Top-ups are not self-service. A callout tells the user to contact the EODH team.

## Usage

The Usage page shows compute usage history for the workspace. It has two views.

**Months view.** A pivot table with one row per month (newest first) and one column per SKU. Columns are sorted by display name, so the order stays the same from month to month. The page gets this from one call to `usage-data?time-aggregation=month`.

**Month detail.** Click a month to open it. The page shows:

- a per-resource total for that month
- every raw usage event in that month, filterable by resource, sortable by resource, value or date, ten per page

The page fetches raw events only for the selected month, and follows the `after` cursor to the end of that month. The code gives the reason. The API cannot return "most recent" events: it pages oldest-first, so an unbounded page of raw events showed very old history for any workspace with real usage. One aggregated call covers all history at low cost, and a one-month range bounds the raw fetch.

**Credits or raw usage.** A toggle switches every figure between credits and raw quantity in the SKU's unit. The default is credits, because credits can be compared across resources. Credit values are what the ledger charged, so they already include any category adjustment and use the pricing policy in force when each event was charged. Small single-event values show extra decimal places so they do not round to `0.00`.

The page has no per-user filter or breakdown.

## Billing accounts

A billing account is the legal and billing relationship between an organisation and EODH. Workspaces are created under an approved billing account. These pages are under Profile because an account belongs to a user, not to a workspace.

| Route | Page | Code |
|---|---|---|
| `/profile/accounts` | Billing accounts list and request form | `src/pages/AccountsPage/AccountsPage.tsx` |
| `/profile/accounts/:accountId` | Account detail | `src/pages/AccountsPage/AccountDetailPage.tsx` |

**List.** `GET /api/accounts` returns the caller's accounts. Each row shows name, status (Pending review, Approved, Request denied), account owner and the names of its workspaces. The name links to the detail page.

**Request a new account.** A dialog collects account name, organisation name, billing address and a reason for opening the account. The user must also agree to the hub terms on behalf of the organisation. The account owner is the signed-in user and is read-only. All fields are required. The form posts to `POST /api/accounts`. The EODH team reviews each request, and the user gets an email when it is approved.

**Detail.** `GET /api/accounts/:id` shows status, owner, billing address, reason for opening, created date, and a table of the account's workspaces with their status.

`accounting-service` has an account-level usage endpoint (`/api/accounts/:id/accounting/usage-data`). The detail page does not use it.

## Open questions

- Compute is priced per CPU-second and per memory-GB-second, charged separately and summed. Is that the right shape, with storage handled some other way?
- Can existing usage and billing records be discarded, or does history need to be carried into the credit model?
- Should the Credits page show the workspace's category and the adjusted rates? That needs an endpoint that returns the category.

## Proposed, not built

These ideas came from the earlier clickable proposal. They are not on `develop`.

- **Usage by user.** Filter and break down usage per member.
- **Budget.** A storage plan (Free, Standard, Plus) billed in pounds, plus a combined compute spending limit with per-member overrides and alerts.
- **Budget policy.** A platform-admin tab listing workspaces near or past their negative-balance limit, and platform-wide default budget settings.
- **Platform costs.** A platform-admin tab listing cloud costs not attributable to one workspace, with a total.
- **Workspace category display.** Show the workspace's pricing category (`standard` or `academic`) on the Credits page.
- **Self-service top-up.** A working "Buy more credits" action.

See [Workspace management UX](workspace-management-ux.md) for the Platform admin area these tabs would sit in, and [Proposed backend endpoints](pending-backend-endpoints.md) for the API work they need.
