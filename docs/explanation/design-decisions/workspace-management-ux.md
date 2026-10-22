---
title: Workspace management UX
doc_status: unreviewed
tags:
  - rc-ui
  - workspaces
last_reviewed:
reviewed_by:
review_notes: "Rewritten 2026-10-21 to describe eodhp-rc-ui develop"
---

# Workspace management UX

Workspace admin tasks used to live in a separate app, `eodhp-workspace-ui`. They are now part of the Resource Catalogue app (`eodhp-rc-ui`), so everything about a workspace is in one place. This page describes the profile and workspace settings pages on the `develop` branch: what each page shows, which API it calls, who can do what, and why where the code or pull requests say so.

All pages call real endpoints, mostly on `eodhp-workspace-services`. There are no mocks. The work arrived in rc-ui #598 to #611 and #613. Accounting pages and billing accounts are covered in [Accounting & billing UX](accounting-billing-ux.md).

## Navigation

The username menu in the top right has these groups:

| Menu group | Entries | Route prefix |
|---|---|---|
| Username (Profile) | Identity, Accounts, Workspaces | `/profile` |
| Workspace settings | Members, Credentials, Provider accounts | `/settings` |
| Accounting / Billing | Credits, Usage | `/accounting` |
| Publishing | Disabled, marked "Soon" | none |
| My data | Purchased data, Workflow outputs, Files, Saved sets | `/my-data` |

Profile, Settings, Accounting and My data each open in a shared sidebar layout (`SidebarNavLayout`). The Settings sidebar also has **S3 credentials**, which is not in the menu. #602 removed the older My Data link from the Catalogue dropdown. The `/my-data/*` routes did not change.

Routes are defined in `src/app/router.tsx`, with each section's pages in `profileRoutes.tsx`, `settingsRoutes.tsx` and `accountingRoutes.tsx`.

### Current workspace

The workspace switcher in the top nav is the single source of truth for the current workspace (`AuthenticationContext.currentWorkspace`). Before #598, several pages had their own workspace picker with separate state. Settings, Accounting and My data pages now read the current workspace from the switcher. The data purchase flow keeps its own workspace dropdown, which does not change the global selection.

Workspace-scoped pages show a workspace badge in the page heading. Profile pages show the signed-in username instead (#608).

## Roles

A workspace has three roles. The backend (`eodhp-workspace-services`) enforces them. The UI shows or hides controls to match.

| Role | Who | Source |
|---|---|---|
| Owner | The owner of the billing account the workspace belongs to | `owner` field on `GET /api/workspaces` |
| Admin | A member explicitly granted admin on that workspace | `GET /api/workspaces/:ws/admins` |
| Member | Anyone in the workspace's member group | `GET /api/workspaces/:ws/users` |

The owner is an implicit admin and is never in the admins list. A user with the `hub_admin` realm role passes every workspace check.

| Action | Member | Admin | Owner |
|---|---|---|---|
| View members and admins | Yes | Yes | Yes |
| Add or remove members | No | Yes | Yes |
| Grant or revoke admin | No | Yes | Yes |
| Link or unlink provider accounts | No | Yes | Yes |
| Delete the workspace | No | Yes | Yes |
| Create and delete own API tokens, request S3 credentials | Yes | Yes | Yes |

The backend refuses to remove the owner from the workspace and refuses to revoke the owner's admin status. There is no way to transfer ownership.

## Profile

### Identity

Route `/profile/identity`. Shows the signed-in user's name, username and email from the Keycloak token. Two buttons open Keycloak's own pages in a new tab: **Edit details** (account page) and **Manage linked accounts** (identity providers). Those pages use Keycloak's styling, not the app's.

Code: `src/pages/IdentityPage/IdentityPage.tsx`.

### Accounts

Route `/profile/accounts`. Lists the user's billing accounts and lets them request a new one. See [Billing accounts](accounting-billing-ux.md#billing-accounts).

### Workspaces

Route `/profile/workspaces`. Lists the user's workspaces from `GET /api/workspaces`, with the billing account each belongs to.

- **Open a workspace.** Click a workspace name to make it the current workspace and go to its Members page (#608).
- **Create.** `POST /api/workspaces` with a name and an approved billing account. The name must be 3 to 50 characters of lowercase letters, numbers and hyphens, and cannot start or end with a hyphen. The Create button is disabled until the user has at least one approved account, and a callout points them to the Accounts page. A 409 response means the name is taken.
- **Delete.** Owners and admins see a Delete button (#613). The dialog warns that all data is removed permanently and needs a confirmation checkbox. It calls `DELETE /api/workspaces/:ws`. The backend removes the workspace asynchronously, so the page drops it from the list at once instead of refetching. If the deleted workspace was current, the app switches to another one. The dialog cannot be closed while the request is running.

To decide who sees Delete, the page fetches the admin list for each workspace the user does not own.

Code: `src/pages/WorkspacesPage/WorkspacesPage.tsx`.

## Workspace settings

### Members

Route `/settings/members`. Lists the current workspace's members with username, name, email and role.

- The owner's row shows an **Owner** badge.
- Owners and admins see a role selector (Member or Admin) on every other row. Each change asks for confirmation in both directions, because granting admin gives real control and revoking removes it (#605). It calls `PUT` or `DELETE /api/workspaces/:ws/admins/:username`.
- Owners and admins can add a member by username (`PUT /api/workspaces/:ws/users/:username`) and remove one after confirmation (`DELETE` on the same path). Removal takes effect at once.
- Other members see the list and their roles as text, with a note that only the owner or an admin can change membership.

Code: `src/pages/MembersPage/MembersPage.tsx`.

### Credentials

Route `/settings/credentials`. Manages DataHub API tokens for the current workspace, ported from the old app (#604).

- List: name, scope, created and expiry (`GET /api/workspaces/:ws/me/tokens`).
- Create: a name and an expiry of 1 to 30 days, default 30 (`POST` on the same path). The token is shown once, with a copy button.
- Delete after confirmation (`DELETE /api/workspaces/:ws/me/tokens/:id`). Anything using the token stops working at once.

Tokens belong to the signed-in user (`me`), so every member manages their own.

Code: `src/pages/CredentialsPage/CredentialsPage.tsx`.

### S3 credentials

Route `/settings/s3-credentials`. Requests temporary AWS credentials for the workspace (`POST /api/workspaces/:ws/me/s3-tokens`). The dialog shows access key ID, secret access key and session token once, each with a copy button, plus the expiry time and a **Copy all to .env** button.

This is a separate page, not a tab on Credentials. #604 gives the reason: S3 credentials are generated on demand and have no list, create or delete lifecycle like DataHub tokens.

The `/me/*` endpoints behind both credential pages do not allow cross-origin calls. In local development the app sends these requests through the Vite `/api` proxy (`src/services/apiUrlUtils.ts`). A deployed build calls `VITE_HUB_BASE_URL` directly.

Code: `src/pages/S3CredentialsPage/S3CredentialsPage.tsx`.

### Provider accounts

Route `/settings/provider-accounts`. Links commercial data-provider accounts so the workspace can order their imagery (#603). Replaces the old app's "Linked accounts" page.

| Provider | How it links | Endpoints |
|---|---|---|
| Airbus | API key, validated, then the user picks a LEGACY and a PNEO optical contract (independent dropdowns, since a workspace can hold one of each). SAR availability is shown. | `POST .../linked-accounts/airbus/validate`, `POST .../linked-accounts`, `DELETE .../linked-accounts/airbus` |
| Planet | API key, validated, then linked | `POST .../linked-accounts/planet/validate`, `POST .../linked-accounts`, `DELETE .../linked-accounts/planet` |
| Open Cosmos | Organisation ID, then sign-in on Open Cosmos's own page (OAuth2 with PKCE). The app stores the session on the backend. | `GET`, `POST`, `DELETE /api/workspaces/:ws/open-cosmos/session` |

All paths are under `/api/workspaces/:ws`. `GET .../linked-accounts` gives the linked status for Airbus and Planet.

The Open Cosmos card reads its status from `GET .../open-cosmos/session`, so "Connected" survives a page reload and follows a workspace switch. **Disconnect** calls `DELETE` and revokes the session on the server. If that call fails, the card stays connected, because the app cannot confirm the session was revoked. The callback checks a `state` parameter, which the old app did not.

Owners and admins can link and unlink. Other members see the cards with the controls disabled and a note explaining why (#605). The backend enforces owner-or-admin for linking, unlinking, key validation and Open Cosmos disconnect. It accepts any member for storing an Open Cosmos session, so the owner-or-admin rule for connecting Open Cosmos is enforced only in the UI.

Code: `src/pages/ProviderAccountsPage/ProviderAccountsPage.tsx`, `src/provider/OpenCosmosAuthProvider.tsx`.

## Open questions

- Should the backend require owner or admin to store an Open Cosmos session, to match the UI and the other provider actions?
- If a workspace owner wants to leave or delete their account, what happens to the workspace? Options include transferring ownership first or promoting an existing admin.

## Proposed, not built

These ideas came from the earlier clickable proposal. They are not on `develop`.

- **Ownership transfer.** Move the Owner role to another member, confirmed by typing their name.
- **Delete my account.** A Profile flow to delete the user's own account, blocked while they own a workspace.
- **Publisher visibility.** A Public or Private control per catalogue collection and workflow, to replace uploading a JSON file. The Publishing menu entry is disabled.
- **Files visibility.** The same Public or Private control per item in workspace file storage.
- **Platform admin.** A hub-admin area outside any workspace, with tabs for workspace settings (category, Dask and GPU toggles), cross-workspace usage, budget policy and platform costs. It needs a "list every workspace" endpoint.
- **Workspace category display.** Show a workspace's pricing category (`standard` or `academic`). The backend can set a category (`PUT /api/workspaces/:ws/category`, hub admin only) but not return it.

For the API work these need, see [Proposed backend endpoints](pending-backend-endpoints.md).
