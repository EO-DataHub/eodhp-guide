---
title: Proposed backend endpoints
doc_status: unreviewed
tags:
  - workspaces
  - rc-ui
last_reviewed:
reviewed_by:
review_notes: "Proposal for client review. Updated 2026-10-21 against shipped code: each row now says whether it is built."
---
# Proposed backend endpoints

A flat, scannable list of every backend endpoint grouped by the frontend pages it is needed by.

This started as a proposal. The Status column says what has shipped as of 2026-10-21: **Built**, **Not built**, or **Built differently**, with a note on how. Paths are as the UI calls them, through the `/api` gateway prefix.

Two different kinds of "admin" appear below — don't conflate them. Owner/admin/member are roles _within one workspace_ (see the Members section). `hub_admin` is a separate, platform-wide role (assigned in Keycloak, not per-workspace) that can act across every workspace on the platform — it's what gates the Workspace category, Workspace integrations, and Platform admin usage endpoints.

## Members — "Members page"

| Method | Path | Request body | Description | Auth | Status |
| ------ | ---- | ------------ | ----------- | ---- | ------ |
| GET | `/api/workspaces/:id/users` | – | Add a `role` field (`owner` / `admin` / `member`) to each member | Any member | Built differently: no `role` field. The UI derives roles from `GET .../admins` and the workspace's `owner` field |
| PUT | `/api/workspaces/:id/users/:username` | – | Adds a member to the workspace | Owner or admin | Built (existed before this proposal) |
| GET | `/api/workspaces/:id/admins` | – | Lists the usernames of explicit admins. The account owner is an implicit admin and is not listed | Any member | Built (`eodhp-workspace-services` PR #53) |
| PUT | `/api/workspaces/:id/admins/:username` | – | Promotes a member to admin | Owner or admin | Built (PR #53). Proposed as owner only |
| DELETE | `/api/workspaces/:id/admins/:username` | – | Demotes an admin back to member; rejects the account owner | Owner or admin | Built (PR #53). Proposed as owner only |
| POST | `/api/workspaces/:id/owner/transfer` | `{ newOwner: username }` | Transfers ownership: sets the new owner and moves the outgoing owner into `admins` | Current owner only | Not built |
| DELETE | `/api/workspaces/:id/users/:username` | – | Removes a member from the workspace and revokes any admin status; rejects if `:username` is the account owner | Owner or admin | Built |
| DELETE | `/api/workspaces/:id` | – | Deletes the workspace | Owner, admin or `hub_admin` | Built (not in the original proposal) |

"Owner" here is the owner of the billing account the workspace belongs to.

## Workspace category — "Workspace category"

| Method | Path | Request body | Description | Auth | Status |
| ------ | ---- | ------------ | ----------- | ---- | ------ |
| GET | `/api/workspaces/:id` | – | Returns the workspace's category | Any member | Built: `category` is a field on the workspace object, left out when no category is set |
| PUT | `/api/workspaces/:id/category` | `{ category: string \| null }` | Sets the workspace's category. `null` clears it, so the workspace pays the default rate | `hub_admin` | Built (`eodhp-workspace-services` PR #56). No dedicated GET |

As built, the allowed values come from the workspace service's configuration (`workspaces.categories`) rather than a fixed `commercial` / `research` pair. They must match the category multipliers in the accounting pricing policy, which today defines `standard` (the default) and `academic`. A value not in the list is a 400, and a non-null value is a 503 when no categories are configured.

## Workspace integrations — "Workspace integrations — Dask, GPU"

| Method | Path                                                                    | Request body                                        | Description                                                             | Auth        | Status |
| ------ | ----------------------------------------------------------------------- | --------------------------------------------------- | ----------------------------------------------------------------------- | ----------- | ------ |
| GET    | `/api/workspaces/:id` (or dedicated `/api/workspaces/:id/integrations`) | –                                                   | Returns whether Dask and GPU integrations are enabled for the workspace | Any member  | Not built |
| PUT    | `/api/workspaces/:id/integrations`                                      | `{ dask_enabled?: boolean, gpu_enabled?: boolean }` | Enables/disables Dask and/or GPU integration for the workspace          | `hub_admin` | Not built |

## Platform admin usage — "Platform admin usage"

| Method | Path | Request body | Description | Auth | Status |
| ------ | ---- | ------------ | ----------- | ---- | ------ |
| GET | `/api/admin/workspaces` | – | Returns every workspace on the platform, regardless of the caller's own membership | `hub_admin` | Not built |
| GET | `/api/accounting/usage-data` | – | Usage across every workspace. Same query as the workspace usage read; `?time-aggregation=day&group-by=workspace` gives one series per workspace | `hub_admin` | Built (`accounting-service` PR #65, not in the original proposal) |

The proposal paired `/api/admin/workspaces` with a per-account usage read. As built, the usage half is served by `GET /api/accounting/usage-data` on its own. To drill down to one workspace, the UI sends the same parameters to `GET /api/workspaces/:id/accounting/usage-data`.

## Profile — "Profile page"

| Method | Path                                                                                   | Request body | Description                                                                                                                                 | Auth                      | Status |
| ------ | -------------------------------------------------------------------------------------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------- | ------ |
| DELETE | account-deletion endpoint — Keycloak itself or a wrapper in front of it (path **TBD**) | –            | Deletes the caller's own account; rejects with the list of workspaces needing an ownership transfer first, if the caller currently owns any | Authenticated user (self) | Not built |

## Publisher — visibility — "Publisher page"

"Collections" are the entries listed under the Publisher page's Catalogue tab; "processes"
are the entries listed under its Workflows tab. Both are STAC terms already used by the
existing catalogue API, not new concepts.

| Method | Path                              | Request body                            | Description                                 | Auth           | Status |
| ------ | --------------------------------- | --------------------------------------- | ------------------------------------------- | -------------- | ------ |
| GET    | `.../catalogs/.../collections`    | –                                       | Add a `visibility` field to each collection | Any member     | Not built |
| PUT    | `.../collections/{id}/visibility` | `{ visibility: 'public' \| 'private' }` | Sets a collection's visibility              | Owner or admin | Not built |
| GET    | `.../catalogs/.../processes`      | –                                       | Add a `visibility` field to each workflow   | Any member     | Not built |
| PUT    | `.../processes/{id}/visibility`   | `{ visibility: 'public' \| 'private' }` | Sets a workflow's visibility                | Owner or admin | Not built |

## Files — visibility — "Files page"

`:storeType` is `object` or `block` — which underlying storage backend the file lives in,
shown as the "Store" column on the Files page.

| Method | Path                                        | Request body                            | Description                           | Auth           | Status |
| ------ | ------------------------------------------- | --------------------------------------- | ------------------------------------- | -------------- | ------ |
| GET    | `/api/workspaces/:id/files`                 | –                                       | Add a `visibility` field to each file | Any member     | Not built |
| PUT    | `.../files/:storeType/:fileName/visibility` | `{ visibility: 'public' \| 'private' }` | Sets a file's visibility              | Owner or admin | Not built |

## Accounting — usage, credits & budgets — "Accounting / Billing — usage, credits & budgets"

The billing backend is `accounting-service`, with routes in `accounting_service/app/app.py`. When this proposal was written it reported usage only in raw quantity and pounds (£). As built, it is credit-based throughout: usage reads report `credits` beside `quantity`, and the price endpoints serve credits per unit. Every endpoint requires a token.

| Method | Path | Request body | Description | Auth | Status |
| ------ | ---- | ------------ | ----------- | ---- | ------ |
| GET | `/api/accounting/pricing-policy` | – | Credit cost per unit for each billable resource (e.g. CPU, GPU), plus the category multipliers and the default category — the per-resource pricing shown on the Credits page | Any signed-in user | Built. A hub admin may pass `?at=` to read an earlier policy |
| GET | `/api/workspaces/:id/accounting/ledger` | – | Proposed: same filters as the usage read but returning `credits` per row/aggregate, for the Usage page | Any member | Built differently: a list of credit transactions (grants, debits, reversals), newest first, filtered by `type`, `start`, `end`, `limit` and `after`. Per-row credits with user, SKU and grouping filters went onto `usage-data` instead (next row) |
| GET | `/api/workspaces/:id/accounting/usage-data` | – | Usage for the workspace, with `credits` beside `quantity`. Takes `start`, `end`, `limit`, `after`, `time-aggregation` (`day`, `week`, `month`, `quarter`), `user`, `sku` and `group-by` | Any member | Built. This is what the Usage page calls |
| GET | `/api/workspaces/:id/accounting/ledger/:transaction` | – | One credit transaction and, for a charge, the arithmetic behind it | Any member | Built (not in the original proposal) |
| GET | `/api/accounting/skus`, `/api/accounting/skus/:sku`, `/api/accounting/prices` | – | Billing items, and the credits per unit for each SKU | Any signed-in user | Built (not in the original proposal) |
| GET | `/api/workspaces/:id/accounting/balance` | – | Workspace's current credit balance, for the Credits page | Any member | Built |
| POST | `/api/workspaces/:id/accounting/balance` | `{ amount: number }` | Tops up/grants credits to the workspace | `hub_admin` only | Not built. Grants are made with the `billing-admin grant` command-line tool |
| GET/PUT | `/api/workspaces/:id/accounting/budget` | `{ enabled: boolean, defaultUserThreshold: number, workspaceThreshold: number }` | Reads/sets one combined spending limit covering CPU + GPU together, matching the Budget page's single "compute" number, plus the default per-user threshold and whether per-user budgets are switched on | Owner | Not built |
| GET/PUT | `/api/workspaces/:id/accounting/budget/users` / `.../budget/users/:username` | `{ username, threshold, alertsEnabled }` | Lists/sets per-user budget overrides, same combined-number shape as the row above | Owner | Not built |

## Other endpoints the UI calls

Not part of the original proposal, but called by the UI on `develop` and served by `eodhp-workspace-services`.

| Method | Path | Request body | Description | Auth | Status |
| ------ | ---- | ------------ | ----------- | ---- | ------ |
| GET/POST/DELETE | `/api/workspaces/:id/open-cosmos/session` | Session (POST) | Reads, stores or removes the workspace's Open Cosmos sign-in session | Any member (GET, POST); owner or admin (DELETE) | Built |
| GET/POST | `/api/accounts` | Account details (POST) | Lists the caller's billing accounts, or requests a new one | Authenticated user | Built |
| GET | `/api/accounts/:id` | – | One billing account | Account owner | Built |

---

## Open questions

Not yet resolved — flagging here rather than cluttering the tables above.

- **Usage visibility** — who can see a workspace's usage data: any member, or admins only? As built, any member can.
- **Member visibility** — who can see the list of workspace members, members or admins only?
- **Promote/demote** — who can promote a member to admin or demote an admin back to member:
  only the owner, or any admin? Answered as built: the owner or any admin.
- **Dask/GPU integrations** — who can enable them: just `hub_admin` for now, or eventually workspace owners/admins too?
- **Visibility changes** — who can change the visibility of a workspace's files, workflows, or catalogue data: owner, admin, or any member?
- **Budget overrides** — who can view or set per-user budget overrides: only the owner, or any admin?
