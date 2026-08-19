---
title: Proposed backend endpoints
doc_status: unreviewed
tags:
  - rc-ui
  - workspaces
last_reviewed:
reviewed_by:
review_notes: "Synced from eodhp-rc-ui/docs/pending-backend-endpoints.md"
---

# Proposed backend endpoints

A flat, scannable list of every backend endpoint grouped by the frontend pages it is needed by.

Two different kinds of "admin" appear below — don't conflate them. Owner/admin/member are roles _within one workspace_ (see the Members section). `hub_admin` is a separate, platform-wide role (assigned in Keycloak, not per-workspace) that can act across every workspace on the platform — it's what gates every endpoint under Platform admin below.

## Members — "Members page"

| Method | Path                                   | Request body             | Description                                                                        | Auth               |
| ------ | -------------------------------------- | ------------------------ | ---------------------------------------------------------------------------------- | ------------------ |
| GET    | `/api/workspaces/:id/users`            | –                        | Add a `role` field (`owner` / `admin` / `member`) to each member                   | Any member         |
| PUT    | `/api/workspaces/:id/admins/:username` | –                        | Promotes a member to admin                                                         | Owner              |
| DELETE | `/api/workspaces/:id/admins/:username` | –                        | Demotes an admin back to member                                                    | Owner              |
| POST   | `/api/workspaces/:id/owner/transfer`   | `{ newOwner: username }` | Transfers ownership: sets the new owner and moves the outgoing owner into `admins` | Current owner only |
| DELETE | `/api/workspaces/:id/users/:username`  | –                        | Removes a member from the workspace; rejects if `:username` is the current owner   | Owner              |

## Platform admin — "Platform admin"

Four tabs, all `hub_admin`-only. All four need a base "every workspace on the platform" endpoint that doesn't exist yet — today's workspace/usage APIs are scoped to the caller's own memberships, or one billing account at a time, not the whole platform:

| Method | Path                    | Request body | Description                                                                                                             | Auth        |
| ------ | ----------------------- | ------------ | ----------------------------------------------------------------------------------------------------------------------- | ----------- |
| GET    | `/api/admin/workspaces` | –            | Returns every workspace on the platform, regardless of the caller's own membership — base list for the three tabs below | `hub_admin` |

### Workspace settings tab

Category and Dask/GPU integrations:

| Method | Path                                                                    | Request body                                        | Description                                                             | Auth        |
| ------ | ----------------------------------------------------------------------- | --------------------------------------------------- | ----------------------------------------------------------------------- | ----------- |
| GET    | `/api/workspaces/:id` (or dedicated `/api/workspaces/:id/category`)     | –                                                   | Returns the workspace's category (`commercial` / `research`)            | Any member  |
| PUT    | `/api/workspaces/:id/category`                                          | `{ category: 'commercial' \| 'research' }`          | Sets the workspace's category                                           | `hub_admin` |
| GET    | `/api/workspaces/:id` (or dedicated `/api/workspaces/:id/integrations`) | –                                                   | Returns whether Dask and GPU integrations are enabled for the workspace | Any member  |
| PUT    | `/api/workspaces/:id/integrations`                                      | `{ dask_enabled?: boolean, gpu_enabled?: boolean }` | Enables/disables Dask and/or GPU integration for the workspace          | `hub_admin` |

### Usage tab

Combined with the existing `GET /api/accounts/:accountId/accounting/usage-data` (see the Accounting group below), the base workspace list above is what powers this tab — no additional endpoint needed.

### Budget policy tab

Entirely new — nothing here exists today:

| Method  | Path                                                                                     | Request body                                                                         | Description                                                                  | Auth        |
| ------- | ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------- | ----------- |
| GET     | `/api/admin/budget-alerts` (or extend the base workspace list with balance/limit/status) | –                                                                                    | Workspaces currently breaching or near their negative-balance limit          | `hub_admin` |
| GET/PUT | `/api/admin/budget-policy/defaults`                                                      | `{ negativeBalanceAllowance: number, computeBudget: number, warnAtCredits: number }` | Platform-wide default budget policy that new workspaces are meant to inherit | `hub_admin` |

### Platform costs tab

Entirely new, and a different data source from everything else in this doc — these are cloud costs, not workspace usage, so this likely comes from a cloud billing/cost-explorer integration rather than the accounting service:

| Method | Path                        | Request body | Description                                                                                   | Auth        |
| ------ | --------------------------- | ------------ | --------------------------------------------------------------------------------------------- | ----------- |
| GET    | `/api/admin/platform-costs` | –            | Cloud costs not attributable to any single workspace, broken down per cost item, with a total | `hub_admin` |

## Profile — "Profile page"

| Method | Path                                                                                   | Request body | Description                                                                                                                                 | Auth                      |
| ------ | -------------------------------------------------------------------------------------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------- |
| DELETE | account-deletion endpoint — Keycloak itself or a wrapper in front of it (path **TBD**) | –            | Deletes the caller's own account; rejects with the list of workspaces needing an ownership transfer first, if the caller currently owns any | Authenticated user (self) |

## Provider accounts — "Provider accounts page"

Airbus/Planet linking already has real, working endpoints (see the codebase's `linkedAccountsService.ts`) — only the two rows below are actually missing.

| Method | Path                                      | Request body | Description                                                                                                                                                                  | Auth  |
| ------ | ----------------------------------------- | ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----- |
| GET    | `/api/workspaces/:id/open-cosmos/session` | –            | Returns whether an Open Cosmos session already exists for this workspace, so the page can show "Connected" correctly on load instead of always starting from "Not connected" | Owner |
| DELETE | `/api/workspaces/:id/open-cosmos/session` | –            | Revokes/deletes the stored Open Cosmos session — today "Disconnect" only clears local state and never reaches the backend                                                    | Owner |

## Publisher — visibility — "Publisher page"

"Collections" are the entries listed under the Publisher page's Catalogue tab; "processes" are the entries listed under its Workflows tab. Both are STAC terms already used by the existing catalogue API, not new concepts.

| Method | Path                              | Request body                            | Description                                 | Auth           |
| ------ | --------------------------------- | --------------------------------------- | ------------------------------------------- | -------------- |
| GET    | `.../catalogs/.../collections`    | –                                       | Add a `visibility` field to each collection | Any member     |
| PUT    | `.../collections/{id}/visibility` | `{ visibility: 'public' \| 'private' }` | Sets a collection's visibility              | Owner or admin |
| GET    | `.../catalogs/.../processes`      | –                                       | Add a `visibility` field to each workflow   | Any member     |
| PUT    | `.../processes/{id}/visibility`   | `{ visibility: 'public' \| 'private' }` | Sets a workflow's visibility                | Owner or admin |

## Files — visibility — "Files page"

`:storeType` is `object` or `block` — which underlying storage backend the file lives in, shown as the "Store" column on the Files page.

| Method | Path                                        | Request body                            | Description                           | Auth           |
| ------ | ------------------------------------------- | --------------------------------------- | ------------------------------------- | -------------- |
| GET    | `/api/workspaces/:id/files`                 | –                                       | Add a `visibility` field to each file | Any member     |
| PUT    | `.../files/:storeType/:fileName/visibility` | `{ visibility: 'public' \| 'private' }` | Sets a file's visibility              | Owner or admin |

## Accounting — usage, credits & budgets — "Accounting / Billing — usage, credits & budgets"

The billing backend (`accounting-service`, routes in `accounting_service/app/app.py`) already has real endpoints for usage and pricing today — but only in raw quantity and pounds (£), with no concept of "credits" at all. Most rows below are credit-based endpoints on top of that; the storage plan row is the exception — it's billed in pounds like today's real endpoints, but has no endpoint at all yet, real or otherwise.

| Method  | Path                                                                         | Request body                                                                     | Description                                                                                                                                                                                                                                                                                                                  | Auth             |
| ------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- |
| GET     | `/api/accounting/pricing-policy`                                             | –                                                                                | Credit cost per unit for each billable resource (CPU, memory, GPU), plus the commercial/research multiplier — the per-resource pricing shown on the Credits page                                                                                                                                                             | Any member       |
| GET     | credit-ledger read endpoint `/api/workspaces/:id/accounting/ledger`          | –                                                                                | Same filters as today's raw-quantity usage endpoint (date range, limit, per-user, aggregation period) but returns `credits` per row/aggregate — real target for the Usage page                                                                                                                                               | Any member       |
| GET     | credit-balance endpoint `/api/workspaces/:id/accounting/balance`             | –                                                                                | Workspace's current credit balance, for the Credits page                                                                                                                                                                                                                                                                     | Any member       |
| POST    | credit top-up/grant endpoint `/api/workspaces/:id/accounting/balance`        | `{ amount: number }`                                                             | Tops up/grants credits to the workspace                                                                                                                                                                                                                                                                                      | `hub_admin` only |
| GET/PUT | `/api/workspaces/:id/accounting/budget`                                      | `{ enabled: boolean, defaultUserThreshold: number, workspaceThreshold: number }` | Reads/sets one combined spending limit covering CPU + memory + GPU together, matching the Budget page's single "compute" number, plus the default per-user threshold and whether per-user budgets are switched on                                                                                                            | Owner            |
| GET/PUT | `/api/workspaces/:id/accounting/budget/users` / `.../budget/users/:username` | `{ username, threshold, alertsEnabled }`                                         | Lists/sets per-user budget overrides, same combined-number shape as the row above                                                                                                                                                                                                                                            | Owner            |
| GET     | `/api/accounting/storage-plans`                                              | –                                                                                | Catalogue of available storage plan tiers (`id`, `label`, `allowanceGb`, `pricePerMonth`) — mirrors the existing `GET /api/accounting/skus` pattern, so tiers/prices can change without a frontend redeploy                                                                                                                  | Any member       |
| GET     | `/api/workspaces/:id/accounting/storage-plan`                                | –                                                                                | The workspace's current plan `planId` plus its current storage usage in GB — note this needs a live storage-size snapshot, not the existing usage-data feed's `EFS-STORAGE-STD`/`AWS-S3-STORAGE` GB-**hours** (a time-averaged historical quantity, not "how much is stored right now")                                      | Any member       |
| PUT     | `/api/workspaces/:id/accounting/storage-plan`                                | `{ planId: 'free' \| 'standard' \| 'plus' }`                                     | Sets the workspace's storage plan tier, for the Budget page's storage section — billed in pounds, separate from the credit endpoints above. Doesn't cover actually charging the monthly fee (a separate payment/invoicing integration), or what happens if usage exceeds the new plan's allowance (not decided anywhere yet) | Owner            |
