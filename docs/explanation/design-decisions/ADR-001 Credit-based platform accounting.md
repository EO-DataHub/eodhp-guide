# ADR-001: Credit-based accounting for platform usage

**Status:** Proposed. Four decisions listed at the end are needed before work starts.
**Date:** 2026-08-19, revised 2026-09-03
**Applies to:** the EO DataHub accounting service. The separate commercial-data purchasing work is unaffected.

## Summary

The platform will measure what each workspace consumes — compute, storage, data transfer — and express it in **credits**: an internal unit, not money. Each workspace holds one shared credit balance. Workspaces can set a spending limit, and the system warns when that limit is approached. Nothing is blocked or switched off.

This gives the platform cost attribution per department and gives grant-holders visibility of their own spending, while deferring anything involving real payments to a later phase.

## Context

The platform records resource consumption today, but only as raw quantities and a price in pounds. There is no notion of a budget, no balance, and nothing that tells a user what they have spent or how much is left.

Two facts shape the design. First, the primary users are academic researchers spending against fixed grants, not anonymous members of the public, so the goal is to help someone avoid overspending rather than to police abuse. Second, the true cost of the underlying AWS services is not yet well understood, so whatever rates are set now will be revised repeatedly as real costs become clear.

Handling real money is deliberately excluded. Payments and invoicing are hard, carry regulatory weight, and are being treated as a separate piece of work.

## Decision

Six choices define the system.

**Credits are an internal unit, not money.** Usage is converted into credits at a published rate. A separate exchange rate records what a credit is worth in pounds, used for reporting and for calibrating rates against real AWS costs. No payment system is involved, and credits are granted administratively.

**Each workspace has one shared balance.** A workspace represents a department or organisation, and its members draw on a single pool. Workspaces can optionally set per-member spending thresholds within that pool.

**Rates are published as one dated policy.** Every rate — the cost of each resource type, the pound exchange rate, and the different multipliers applied to commercial and research workspaces — is versioned together as a single set. Each charge records which version priced it, so a charge from six months ago can still be explained exactly, even after rates change.

**The record is never edited.** Charges accumulate in a ledger that works like a bank statement: entries are added, never altered or removed. A mistake is fixed by adding a correcting entry, so the original record of what a user was told at the time survives. This matters for auditing and for correcting metering errors, which typically affect thousands of charges at once.

**Budgets warn; they do not block.** When a workspace approaches its limit, the system records the fact and announces it. No work is prevented, no job is stopped, and no service waits on the accounting system before starting work. Overspending remains possible, by design.

**Pricing mistakes can be corrected retrospectively.** If a rate is found to be wrong after charges have been made under it, those charges can be recalculated under a corrected version, with both the original and the correction preserved.

## Excluded from this phase

| Excluded | Consequence |
|---|---|
| Payments, invoicing, refunds | Credits stay notional. Granting credit is an administrative action |
| Blocking or throttling on overspend | A workspace can exceed its budget. The system reports it and nothing more |
| Delivering the warnings | See risk 2 below. The warning is produced, but nothing yet sends it to anyone |
| GPU measurement | See risk 1 below. Owned by a different component |

## Consequences

**What this makes possible.** Every charge can be explained months later, including which rates and which workspace category produced it. Rates can be re-tuned as often as needed without corrupting historical records. Cost can be attributed per department, per member, and per resource type. When payments do arrive, they attach to an existing ledger rather than requiring one to be built.

**What it costs.** The accounting service gains a substantial amount of new structure, and for the first time it will send messages to other parts of the platform rather than only receiving them. Database changes will be managed with a proper migration tool, which is being introduced as part of this work — a small addition that prevents a class of deployment failure the service is currently exposed to.

**What it does not do.** Nobody is stopped from overspending. A workspace that goes over its budget is recorded and reported, and depends on someone acting on that report.

## Effort

The work was originally scoped as fifteen items with a first-pass estimate, of which one — the credit balance model — carried a single 20-day estimate, far larger than anything else.

That item has been split into three smaller pieces, and the technical risk it carried has been removed rather than merely divided. The original concern was many users spending from one balance at the same time and the balance becoming wrong. Because the ledger is only ever added to, that failure cannot occur.

Four other items reduced in size once the existing code was examined more closely; three new items were added, covering corrections, retrospective re-pricing, and publishing the warnings. **The overall total is roughly unchanged.** The reductions and additions offset each other, and the largest single risk in the original estimate is gone.

## Risks

**1. GPU usage is not measured anywhere, and two planned pages depend on it.** The frontend design prices GPU time and treats it as a budgeted cost, but nothing on the platform currently measures GPU consumption. Adding it sits with a different component and a different team. Until it exists, those pages cannot show correct figures. This is the largest external dependency.

**2. Budget warnings will not reach anyone until a notification system exists.** The accounting service will detect a breach and announce it, but no subsystem yet listens for that announcement, and none is designed. The frontend design already offers users a switch to turn alerts on. Unless the listening side is funded, workspaces will be able to configure budgets and enable alerts that are recorded but never delivered. This is the risk most likely to surprise stakeholders, because the feature will appear to work.

**3. A shared message contract between two services has already drifted apart.** Two services exchange workspace information, and their definitions of that message have quietly diverged — different field names on each side. It causes no failure today only because the mismatched fields happen not to be read. This work needs to add a field to that same message, which means changing two services, written in two different languages, in step, and fixing the existing mismatch at the same time. It is straightforward work but it crosses a team boundary.

**4. The missing permission level is being built, and one piece of it is still needed.** The wider frontend design uses three levels within a workspace: owner, administrator, member. The platform had only two when this ADR was written. A change now under review in the workspace service adds the administrator level, which removes this risk for the frontend work as a whole.

One small addition is still required before the accounting service can read that level. The workspace service can currently answer "who administers this workspace" but not "which workspaces does this person administer", and the second form is what a login token needs. The request has been made on the change under review. It is half a day of work, but it crosses two repositories and a change to the identity system, so it needs scheduling rather than assuming.

A decision follows from this. Administrators can already manage a workspace's members and its linked billing accounts. Whether they should also be able to set the workspace's spending limit is a question for whoever owns that policy. The accounting design works either way and the switch is one value per endpoint.

**5. The accounting service trusts the identity in a request without checking it.** Every request carries a login token stating who the user is and what they are entitled to. The service reads that token but does not check the cryptographic signature which proves it is genuine. It relies on the platform gateway having done so before the request arrives.

That has been reasonable while the service only answered questions. It reports what a workspace has consumed, and a forged token would reveal usage figures and nothing else. The credit features change what a token can do: the same claims will decide who may set a workspace's spending limit and who may grant credits.

Two things close this. First, confirm that nothing inside the platform can reach the accounting service except through the gateway — the reliance is on that being true, and it has never been written down. Second, add signature checking to the service, estimated at a day and a half and included in the figures above. A working example already exists in another service on the platform, so this is a known quantity rather than research.

The recommendation is to do both before the administrative features reach production, and to treat the read-only features already planned as acceptable in the meantime.

## Decisions needed

| # | Decision | Why it matters |
|---|---|---|
| 1 | Is GPU measurement funded and scheduled, and by whom? | Two planned pages cannot show correct figures without it. It sits outside this work |
| 2 | Is a notification system funded? | Without it, budget warnings are recorded and never delivered, and the alerts switch offered to users does nothing |
| 3 | Confirm that credits stay notional for this phase, with no money movement | The design assumes it throughout. Reversing this assumption later is expensive; assuming it now is not |
| 4 | Confirm that the accounting service cannot be reached except through the platform gateway | The service trusts login tokens without checking them, on the assumption that the gateway checks first. See risk 5. If the assumption does not hold, the fix moves from hardening to urgent |

One decision listed in an earlier version of this record has been settled and acted on: existing accounting records are preserved rather than the database being recreated. Both deployed databases have been migrated in place.

Two further questions need answers before the relevant pages are built, though neither blocks development:

- Whether a workspace's usage figures are visible to every member or only to administrators. The frontend proposal currently states both.
- Whether a workspace administrator, as well as the owner, may set the workspace's spending limit. See risk 4.

## Where the detail lives

- **Credits ledger design decisions** — the twelve technical decisions behind this ADR, with reasoning
- **Credits ledger schema** — the database design
- **Credits ledger scoping** — the task list and estimates, reworked to match the decisions recorded here
