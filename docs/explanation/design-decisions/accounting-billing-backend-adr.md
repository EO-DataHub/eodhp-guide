# ADR-001: Credit-based accounting for platform usage

- **Status:** Proposed. Four decisions listed at the end are needed before work starts.
- **Date:** 2026-08-19
- **Applies to:** the EO DataHub accounting service.

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

| Excluded                            | Consequence                                                                   |
| ----------------------------------- | ----------------------------------------------------------------------------- |
| Payments, invoicing, refunds        | Credits stay notional. Granting credit is an administrative action            |
| Blocking or throttling on overspend | A workspace can exceed its budget. The system reports it and nothing more     |
| Delivering the warnings             | See risk 2 below. The warning is produced, but nothing yet sends it to anyone |
| GPU measurement                     | See risk 1 below. Owned by a different component                              |

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

**4. The frontend assumes a permission level that does not exist.** The wider frontend design uses three levels within a workspace — owner, administrator, member — but the platform currently has only two. The accounting features work with the two that exist and have been built so the third can be admitted later by changing a single value. The rest of the frontend work will need the missing level built.
