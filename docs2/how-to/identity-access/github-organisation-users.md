---
title: How to Manage Users in the EODH GitHub Organisation
doc_status: ok
last_reviewed: 2026-05-20
reviewed_by: geodowd
review_notes: "Copied from docs/EO-DataHub GitHub Organisation User Management Guide.md"
---

# How to Manage Users in the EODH GitHub Organisation

**Organisation:** [github.com/EO-DataHub](https://github.com/EO-DataHub)  
**Requires:** Owner or team Maintainer role (see [Permission Levels](#permission-levels) below)

---

## Overview

The EO Data Hub GitHub org uses **teams** to manage access. Rather than granting repo access person-by-person, users are placed into a team that already has the right permissions. This means:

- Adding a new person from a known supplier → add them to that supplier's team
- Granting access to a new supplier → create a new team, configure its permissions, then add members
- Removing someone → remove them from the org (or just from their team)

---

## Permission Levels

| Role | Who has it | Can do |
|---|---|---|
| **Owner** | Alex, Gemma (and a small number of others) | Everything, including deleting the org — keep to a minimum |
| **Member** | All invited users | Base-level access set in org Settings → Member Privileges |
| **Team Maintainer** | Designated lead per team | Add/remove members within their own team |
| **Outside Collaborator** | Users added directly to a repo (not via team) | Access only to the specific repos they were granted |

> ⚠️ Owners can delete the entire organisation. Only grant owner access when genuinely necessary.

---

## Scenario 1: Adding a New Person from an Existing Supplier

This is the most common case (e.g. A supplier onboards a new developer).

### What you need first

Ask the person for their **GitHub username** (not their name or email — usernames are unambiguous and avoid accidentally inviting the wrong person).

### Steps

1. Go to [github.com/EO-DataHub](https://github.com/EO-DataHub) → **Teams** tab
2. Find the relevant supplier team (e.g. `supplier-name-1`, `supplier-name-2`)
3. Click **Add a member**
4. Type their GitHub username and select them
5. If they are not yet in the org, GitHub will prompt: _"Would you like to invite them to the organisation?"_ — confirm yes
6. The user will receive an **email invitation** which they must accept within **7 days**
7. Until accepted, their status shows as **Pending** — they cannot access anything yet

> 💡 If the invitation expires, simply repeat the process to re-send it.

---

## Scenario 2: Adding a New Supplier (New Team)

When a new organisation/supplier joins the project and multiple people need access.

### Steps

1. Go to [github.com/EO-DataHub](https://github.com/EO-DataHub) → **Teams** tab → **New team**
2. Name the team in lowercase with hyphens, no spaces (e.g. `new-supplier-dev`)
   - If the supplier has contractors needing narrower access, consider creating a second nested team (e.g. `new-supplier-readonly`)
3. Set a **Team Maintainer** from the supplier side — they can then manage their own team membership (but cannot invite people to the org themselves)
4. Configure the team's **org-wide role** if appropriate (see below)
5. Add the team to specific **repositories** if they don't need org-wide access
6. Add members using the process in Scenario 1

### Choosing the right permission level

| Supplier needs to… | Recommended role |
|---|---|
| See all repos and raise issues | `All repository triage` |
| Work across all repos (push, PRs, branch management) | `All repository maintain` |
| Only work on 1–2 specific repos | Add team directly to those repos only |
| Full repo admin (rare) | `All repository admin` — use temporarily and remove after |

> 📌 Prefer granting the **minimum necessary access**. Over-permissioning creates risk; under-permissioning creates support burden. For trusted core suppliers working broadly across the platform, `All repository maintain` is usually the right default.

---

## Scenario 3: Granting Access to a Specific Repository

For a user or team that only needs access to one or a few repos (e.g. a contractor fixing a single bug).

### For an individual

1. Navigate to the repository → **Settings** → **Collaborators and teams**
2. Click **Add people** and enter their GitHub username
3. Choose the permission level (`Read`, `Triage`, `Write`, `Maintain`, `Admin`)
4. They become an **Outside Collaborator** (not a full org member)

> ⚠️ Outside collaborators cannot be added to teams. If someone ends up needing access to more repos over time, move them to a full org membership via a team instead.

### For a team

1. Navigate to the repository → **Settings** → **Collaborators and teams**
2. Click **Add teams**, search for the team name, and set permission level
3. All current and future members of that team inherit this access

---

## Scenario 4: Removing a User or Offboarding a Supplier

### Remove an individual

1. Go to org **Settings** → **People**
2. Find the user, click the `...` menu → **Remove from organisation**
3. This immediately revokes all access — team memberships and repo permissions are all removed

Alternatively, convert them to an **Outside Collaborator** if they should retain access to specific repos they were directly granted (e.g. a contractor who finished most work but still needs one repo).

### Offboard a supplier team

1. Remove all members from the team individually (or remove each from the org), **or**
2. Delete the team entirely (repo access granted via that team is revoked for all members)

> 💡 If the same role will be taken over by a new supplier later, consider **renaming** the team rather than deleting it (e.g. rename `supplier-name` to `hub-platform-maintainers`). The repo/project permissions stay intact and you just swap the members.

---

## Auditing Access

To check what a specific user has access to and why:

1. Go to org **Settings** → **People**
2. Click on the person's **name** (not their avatar — that goes to their public profile)
3. You'll see a breakdown of every repo they can access and the reason (team membership, direct grant, org role, etc.)

This is useful for:
- Verifying access before/after changes
- Investigating unexpected access
- Periodic membership reviews

---

## GitHub Projects (Tracker Boards)

GitHub Projects live at the **org level** and have their own access controls separate from repos and teams.

- Private projects require **explicit access** — org membership alone is not enough
- The easiest way to grant project access is to **add a team to the project** (Settings inside the project → Manage access → Add team)
- Then adding someone to that team grants them both repo and project access in one step

> ⚠️ You cannot add someone to a project directly unless they are already an org member. Always invite them to the org (via a team) first.

---

## Quick Reference: Most Common Task

**"A new developer from Supplier X is joining — what do I do?"**

1. Ask them for their GitHub username
2. Go to Teams → find their supplier's team → Add a member → enter username → confirm org invite
3. Tell them to check their email and accept the invite within 7 days
4. Done — they inherit all the permissions their team has

---

## Further Reading

- [GitHub Docs: Inviting users to your organisation](https://docs.github.com/en/organizations/managing-membership-in-your-organization/inviting-users-to-join-your-organization)
- [GitHub Docs: Roles in an organisation](https://docs.github.com/en/organizations/managing-peoples-access-to-your-organization-with-roles/roles-in-an-organization)
- [GitHub Docs: Managing team access to a repository](https://docs.github.com/en/organizations/managing-user-access-to-your-organizations-repositories/managing-team-access-to-an-organization-repository)
- [GitHub Docs: Outside collaborators](https://docs.github.com/en/organizations/managing-user-access-to-your-organizations-repositories/managing-outside-collaborators)
- [GitHub Docs: Organisation Projects access](https://docs.github.com/en/issues/planning-and-tracking-with-projects/managing-your-project/managing-access-to-your-projects)
