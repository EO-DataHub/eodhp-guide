---
title: Workspace management UX
doc_status: unreviewed
tags:
  - rc-ui
  - workspaces
last_reviewed:
reviewed_by:
review_notes: "New page summarising feat/add-workspace-services for client review"
---

# Workspace management UX

Today, workspace admin tasks — members, credentials, billing, publishing — live in a separate app (`eodhp-workspace-ui`), disconnected from the main Resource Catalogue app. We're folding that functionality into the Resource Catalogue app itself, so everything about a workspace lives in one place instead of two. This page walks through what that looks like so far, on branch `feat/add-workspace-services`.

Two labels appear throughout this page:

- **Rebuilt** — this already existed in the old workspace app; we've reproduced the same capability here, usually with a better UI.
- **Draft** — this is a new proposal. Nothing behind it is wired up to a real backend yet — it's a clickable mock so you can react to the idea before we build it for real.

Where a section is a draft, it ends with specific questions we'd like your view on.

## Settings navigation — Rebuilt, reorganised

Workspace settings (Members, Credentials) and the publishing tools are now reachable from inside the Resource Catalogue app itself, from the username menu in the top right, rather than linking out to a separate app. Settings, Accounting/Billing, Publishing, and My Data each open into a sidebar layout, similar to how the old workspace app organised its own menu.

**Question for you:** does grouping things this way (Settings / Accounting & Billing / Publishing / My Data) match how you'd expect to find these, or would you group them differently?

## Members — mixed: list is rebuilt, roles are a draft

**Rebuilt:** the member list itself — who's in a workspace, and removing a member — carries over the same capability the old app had.

**Draft:** the old app only ever had one distinction: the workspace owner, and everyone else ("member"). We've mocked up a proper three-tier model — **Owner** (one per workspace, transferable), **Admin** (can be promoted/demoted by the owner), and **Member** — plus an ownership-transfer flow (type the new owner's name to confirm). None of this is saved anywhere real yet; it resets if you refresh the page.

**Questions for you:**

- Who should be able to see workspace members — every member, or just admins/owner?
- Who should be able to promote a member to admin or demote them back — only the owner, or any admin too?

## Credentials — Rebuilt

API tokens for connecting external tools to a workspace — create, view, and revoke — carried over largely as-is from the old app, restyled to match the rest of the Resource Catalogue app. This is real, not a draft, but there is a CORS issue with it that means it isn't working on the server.

## Provider accounts — Rebuilt, with two known gaps

A new "Provider accounts" tab in Settings, next to Members and Credentials, lets a workspace link commercial data-provider accounts so it can order their imagery — carried over from the old app's "Linked accounts" page. Airbus and Planet linking (a simple API key) is a straight rebuild of what already existed and works for real today. Open Cosmos (sign in via their own login page) is also real, not a mock, but has two gaps carried over from the old app that need backend work to close: the "Connected" status doesn't survive a page reload even if a session already exists, and "Disconnect" only clears the button's own state rather than actually revoking anything server-side. Both are tracked in [Proposed backend endpoints](pending-backend-endpoints.md).

**Question for you:** same as Members — who should be able to link/unlink these accounts: only the workspace owner (current behaviour), or any admin too?

## Profile page - Draft

Clicking on the users username in the menu takes them to a page where there are deep-links out to Keycloak's own account page for changing your name/email, and to Keycloak's linked-accounts page.

The Profile page also has a **draft** "delete my account" flow, blocked if you currently own any workspace (since deleting an owner's account with no owner left behind is unresolved). No account is actually deleted by this yet.

**Questions for you:**

- If a workspace owner wants to leave/delete their account, should we require them to transfer ownership first (current mock), or offer something else — e.g. automatically promote an existing admin to owner?
- Is it ok that the pages to manage account details and identity providers are in keycloak and so look different to the rest of the site?

## Publisher & Files — visibility, Draft

Both the Publisher page (for catalogue collections and workflows) and the Files page (workspace file storage) now have a Public/Private visibility control per item, gated behind a confirmation step. Nothing is actually made public yet — this is UI only, to check the interaction feels right before we build the real permission checks behind it. This is designed to replace the current publishing workflow where the user has to upload a json file in a particular format.

**Question for you:** who should be allowed to change an item's visibility — only the workspace owner, any admin, or any member?

Also worth flagging: visibility is currently a simple binary (Public/Private). If you need anything more granular later — e.g. shared with specific other workspaces — that's a bigger piece of work than what's mocked here, so it's worth saying now if that's a requirement.

## Platform admin — Draft

Platform/hub admins now have their own area, separate from any single workspace's Settings — reachable from the username menu, visible only to platform admins. It exists because the workspace switcher only ever lists workspaces you're a member of, and a platform admin needs to reach workspaces they don't belong to at all (e.g. to enable GPU access for a workspace they're not in). It has four tabs; two are covered here, two are billing concepts covered in [Accounting & billing UX](accounting-billing-ux.md).

- **Workspace settings** — every workspace on the platform, with its **Category** (Commercial/Research — drives the pricing multiplier, see the billing doc) and **Dask/GPU integration** toggles.
- **Usage** — the same cross-workspace usage leaderboard; every workspace's usage, for spotting which are consuming the most.
- **Budget policy** and **Platform costs** — see the billing doc.

All four tabs are mock data — the real version needs a "list every workspace" endpoint that doesn't exist yet (today's workspace/usage APIs are scoped to the caller's own memberships, or one billing account at a time, not the whole platform).

**Questions for you:**

- Should platform admins be the only ones who can change a workspace's category or enable Dask/GPU, or should workspace owners eventually be able to self-serve some of this?
- Beyond "who's using the most," is there other information you'd want on the Usage tab — e.g. flagging workspaces close to a budget limit, or inactive workspaces?

---

For the underlying API work needed to make the draft items real, see [Proposed backend endpoints](pending-backend-endpoints.md).
