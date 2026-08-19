---
title: Accounting & billing UX
doc_status: unreviewed
tags:
  - rc-ui
  - workspaces
last_reviewed:
reviewed_by:
review_notes: "New page summarising feat/add-workspace-services for client review"
---

# Accounting & billing UX

The old workspace app billed everything in raw pounds, shown as a flat list of invoices. The proposal here moves to a shared **credit** currency: a workspace holds a credit balance, spends it on compute (CPU/memory/GPU) at a per-resource rate, and tops it up as needed. This page covers three new pages built around that idea — Credits, Usage, and Budget — plus the workspace category setting that drives pricing. All of it is a **draft**: nothing here is connected to a real backend yet, since the credit-ledger model itself doesn't exist in the billing system today. It's a clickable proposal, built so you can react to the idea before real engineering work starts.

## Credits

Shows the workspace's current credit balance, a "burn rate" table (how many credits each resource costs — e.g. 1 credit/hour for CPU, 0.5 credits/GB-hour for memory, 10 credits/hour for GPU), and a "Buy more credits" action mocked at the moment. Memory has its own rate rather than being bundled into the CPU rate because the real usage-data API already meters it as its own separate quantity, alongside CPU. It also shows the workspace's category (Commercial/Research) read-only, since it drives the pricing multiplier — the editable control lives with the other platform-admin-only, cross-workspace settings, not here — see [Workspace management UX](workspace-management-ux.md).

**Questions for you:**

- This assumes the CPU/memory/GPU credit-rate model is the right shape, with storage and data egress billed as a storage plan (see budget section) — is this ok?
- The formula behind the burn-rate table is **additive, not multiplicative**: `cost = (cpu-seconds × cpu-rate) + (memory-gb-seconds × memory-rate)` — CPU and memory are priced and charged independently, then summed, mirroring how the real usage-data API already meters them as two separate quantities. A job that's light on one resource and heavy on the other still gets charged fairly for whichever it actually uses. Are you happy with cost being calculated this way?

## Usage

Shows consumption over time, filterable by month and by user, with a per-user breakdown, broken down by resource type (CPU, memory, GPU). This is the page real usage data already partially supports today (the underlying usage API exists, just not denominated in credits yet) — CPU and memory are already separately metered there today, which is what this page's resource breakdown mirrors.

The "by user" functionality won't be available until the next phase, but we decided to include it in the interface design now so it can be easily plugged in later.

## Budget

Two separate things, worth noting as separate because they work differently:

- **Storage** is still a flat monthly plan (Free / Standard / Plus), billed in pounds, separate from the credit pool. Unlike the compute side, there's no proposed endpoint for this yet at all — reading a workspace's plan/usage and changing plan are both entirely mocked, with nothing plumbed in.
- **Compute spending limits** are new: one combined limit covering CPU + GPU together (not split per resource), plus optional per-member overrides with alerts.

The "by user" functionality won't be available until the next phase, but we decided to include it in the interface design now so it can be easily plugged in later.

**Questions for you:**

- Does keeping storage as a separate flat-fee plan (rather than folding it into the credit pool like compute) match your expectations, or should storage eventually be metered the same way?
- Who should be able to set per-member budget overrides — only the workspace owner, or any admin?

## Budget policy

A new tab under [Platform admin](workspace-management-ux.md#platform-admin-draft) (platform admins only), with two parts:

- **Budget alerts** — a cross-workspace list of workspaces that have breached or are near their negative-balance limit, so a platform admin can spot problems without checking every workspace one by one.
- **Default budget policy** — platform-wide defaults (negative balance allowance, compute budget, warn-at threshold) that new workspaces are meant to inherit until a workspace owner sets their own override on the Budget page.

Both are mock data — there's no real cross-workspace alerting yet, and new workspaces don't actually inherit these defaults yet either.

**Question for you:** who should be notified when a workspace breaches its limit — just the platform admin shown here, or the workspace's own owner/admins too?

## Platform costs

A new tab under [Platform admin](workspace-management-ux.md#platform-admin-draft) listing cloud costs that aren't attributable to any single workspace (core services, logging & monitoring, cataloguing & harvesting), with a total. These are platform-wide overheads, not something any one workspace is billed for.

**Question for you:** is a flat list with a total enough, or would you want these costs broken down over time (e.g. month by month) to spot trends? Or is using the AWS cost-explorer functionality good enough?

---

For the underlying API work needed to make any of this real, see [Proposed backend endpoints](pending-backend-endpoints.md).
