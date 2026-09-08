# Credits and ledger — design decisions

This note records the design decisions taken for the credits, ledger, and budget features in `accounting-service`, and what each one changes in the task list held in [Credits ledger scoping](Credits%20ledger%20scoping.md). It covers metering of a workspace's own platform usage (compute, storage, object-store calls and transfer). The commercial-data purchasing rearchitecture is a separate piece of work and is not affected.

Decisions are numbered for reference. Each states what was decided and why, because the reasons constrain later choices more than the decisions themselves do.

## 1. Credit pools belong to workspaces

A credit pool belongs to a workspace, and the workspace's member users share it.

A workspace is an organisation or a department. An `account` in `eodhp-workspace-services` is a user's billing registration, not an organisation, so it is the wrong home for a shared pool. Membership of a workspace is a flat Keycloak group whose name is the workspace name (`eodhp-workspace-services/api/services/workspaces.go:218`).

Four authorisation tiers exist once PR 53 on `eodhp-workspace-services` lands. The credit features reuse them and add no roles of their own:

| Tier | How it is established | Credit permissions |
|---|---|---|
| `hub_admin` | Keycloak realm role, short-circuits all checks (`api/services/utils.go:68`) | Grant credits, issue reversals, re-price history, set a workspace's category |
| Workspace owner | The `workspaces-owned` JWT claim, which `accounting-service` already reads (`app/app.py:178-199`). The Go service derives the same fact from account ownership (`IsUserAccountOwner`, `api/services/utils.go:88`) | Configure the workspace's budget and thresholds |
| Workspace admin | The `workspace_admins` table in `eodhp-workspace-services`, reaching this service as a `workspaces-admin` JWT claim (D11) | Undecided. See D11 |
| Member | In the workspace's Keycloak group | Read balance and usage |

Before PR 53, a workspace admin was inferred from account ownership, so a second admin, or an admin who was not the account owner, could not be expressed at all. PR 53 records admin status explicitly in a `workspace_admins` table and generalises `isUserWorkspaceAuthorized` to accept admins as well as owners. Admins gain member management and linked-account management. The account owner stays an implicit, permanent admin, and admin status is revoked automatically when a user's workspace membership is removed.

## 2. Credits are separate from money

Credits are an internal quota unit. Nothing in this work moves real money, and the payment and invoicing system stays a separate future piece of work.

The versioned credit-to-currency exchange rate is therefore a reporting and calibration value, not a funding conversion. It answers "this workspace's usage was worth £Y" while the platform learns what AWS actually costs. It is not read when a pool is funded.

The ledger stays money-agnostic so the payment work can attach later without reshaping it. Credit grants are an administrative action by a `hub_admin`.

## 3. A pricing policy is a single versioned bundle

A pricing policy is one versioned record covering every rate at once: the per-resource-type credit consumption rates, the credit-to-currency exchange rate, and the per-category multipliers. It replaces the existing per-SKU approach, where each `BillingItemPrice` row carries its own `valid_from`/`valid_until` (`accounting_service/models.py:155`).

Rates will be re-tuned repeatedly as real AWS costs become clearer. A calibration pass changes many rates together, so the set of numbers in force at a given time is the thing worth versioning. Under the per-SKU model, "what was in force last March" means reconstructing it from many independent date ranges that only coincidentally align, and nothing enforces that they do.

The config loader gains a matching job: each load either matches the current policy or mints a new version.

This collapses four task-list items into one mechanism — the exchange rate from task 0 joins the policy, and tasks 3, 7, and 8 become operations on a single version identifier.

## 4. Enforcement is eventually consistent, and warns only

When a workspace crosses a threshold, `accounting-service` publishes a breach message to Pulsar. Nothing is blocked. No caller waits on `accounting-service` before starting work.

The primary users are academics spending against grant budgets, not anonymous members of the public, so the goal is to help someone avoid overspending rather than to prevent abuse. Two details of the original requirements point the same way: a *negative*-balance limit accepts overshoot by construction, and the pre-execution cost estimate (task 10) is advisory. Integrity effort belongs on metering accuracy, not on fraud prevention.

A synchronous gate was rejected because it makes `accounting-service` a hard dependency of all compute provisioning, with an unattractive choice when it is unavailable: fail open and overspend, or fail closed and halt the platform.

The Pulsar seam is built now even though nothing blocks. Adding "refuse new provisioning" later means writing a consumer, not reworking the ledger.

## 5. One budget per workspace, with optional per-user thresholds

A workspace has one configurable negative-balance limit. Every SKU draws on the same pool. On top of that, a workspace may set a default per-user threshold and per-user overrides, and switch per-user budgets off entirely.

**This supersedes the earlier decision to hold limits per resource type.** That decision was taken to keep the information-preserving direction open, on the reasoning that collapsing many limits into one is easier than splitting one into many. The frontend design settled on a single workspace figure, so the collapse is now being made — in the direction that was deliberately kept cheap.

Per-type *attribution* survives regardless, because ledger rows carry `sku`. Per-type reporting and filtering (task 12) are unaffected. What is given up is per-type *limits*, and with them the ability to warn that one resource class is consuming a budget intended for another.

The budget key becomes `(workspace, user-or-null)`, where the null-user row is the workspace-wide limit and the others are per-user overrides. No concept of grouping SKUs is needed: an earlier reconciliation considered grouping CPU and GPU into a "compute" budget, and a single pool removes the requirement entirely.

## 6. Category multipliers are policy; category assignment is operational fact

The multiplier for each category (commercial, research) lives in the versioned pricing policy. The assignment of a workspace to a category comes from `eodhp-workspace-services` over the existing `workspace-settings` Pulsar channel, with a default for newly created workspaces, which are self-service and so exist before anyone categorises them.

The two change for different reasons. Re-tuning the research multiplier is a calibration pass that should apply to every research workspace from that point. Recategorising one department says nothing about pricing and must not mint a policy version, because that would imply every rate changed.

**Each ledger transaction records the resolved category alongside the policy version.** A row reading "priced under policy v7, category=research" stays explainable after the workspace's category changes, with no category-history table and no question about re-pricing. Without it, explaining an old charge means reading today's category and computing a different, wrong number.

Category changes are expected to be rare — spinning up a new workspace is more likely than converting one — but the capability is retained deliberately. It is available to `hub_admin` only and is not exposed on the workspace-admin API surface.

Assignment is always a manual `hub_admin` action, taken case by case once the workspace exists. A workspace therefore has no category for however long that takes, and usage in that window is priced under the policy's `default_category`. This is accepted rather than prevented. Credits carry no financial consequence yet (D2), and if such a window ever matters, each row records the category it was priced under, so the re-pricing machinery in D8 can correct it retroactively.

## 7. Corrections are append-only, and support bulk

A wrong charge is undone by a new compensating row, never by editing or deleting a ledger row. A reversal is a typed transaction that references the original transaction's UUID and carries a correction-batch identifier.

The expected common case is a metering bug affecting thousands of rows at once, not a single disputed job. Without a batch identifier, such a correction becomes thousands of unlinked rows and no way to answer what was fixed or when.

A reversal reuses the **original** policy version. Fixing a wrong quantity is not re-pricing, and picking up current rates would silently merge a correction with a calibration change.

Only a `hub_admin` may issue a reversal.

## 8. Historical re-pricing is in scope

Correcting a policy and re-pricing the usage already charged under it is supported, though expected to be rare. This needs a "corrected by" relationship between policy versions and a re-price runner over a date range.

It reuses decision 7's correction-batch machinery: for each affected row, read its quantity and category, apply the corrected policy, then write a reversal and a fresh charge in one batch.

This is possible only because of decisions 3 and 6. Re-pricing needs quantity, category, and policy version on every row. Had the ledger stored the computed credit amount alone, historical re-pricing would be impossible. Task 8's requirement for "a reference, not a computed cost" protects exactly this.

## 9. Notification delivery is out of scope

`accounting-service` publishes a threshold-breach message stating the fact: workspace, resource type, threshold, balance, and timestamp. It does not send email and does not resolve recipients.

**No subsystem consumes this message yet.** The message is fired into the void, and something must be written to catch it. That subsystem is undesigned and outside this piece of work. Until it exists, warn-only enforcement has no visible effect on a user, which matters because the notification *is* the control.

Recipient resolution belongs to that future consumer. `accounting-service` holds no user identities or email addresses; Keycloak and `eodhp-workspace-services` do. `eodhp-workspace-services` already sends email through AWS SES for account approval and denial (`api/services/accounts.go:314-406`), which is the nearest existing precedent.

This makes `accounting-service` a Pulsar producer for the first time. It is consume-only today: every `process_payload` in `accounting_service/ingester/messager.py` returns an empty action list, and the service holds no producer. The breach message therefore adds producer configuration, a topic in the Pulsar deployment, and a message schema. The schema belongs with the others in `eodhp-utils`, which every pipeline component depends on, so the addition is a change to a shared contract library rather than a change local to this service.

## 10. Usage with no applicable policy is priced under the earliest policy

When a `BillingEvent` carries an `occurred_at` that no policy's validity range covers, it is priced under the policy with the earliest `valid_from`.

This arises when an event is backfilled from before the first policy existed. The alternatives are to skip the event, which loses a charge silently, or to park it for retry, which needs a queue and a way to notice that queue filling up. Pricing under the earliest policy keeps the ledger complete, and the resulting row records which policy it used like every other row.

The case is expected to be rare. It is specified because the pricing engine needs defined behaviour for it, and an unspecified branch tends to become a silent skip.

## 11. Authorisation is expressed as ordered tiers

Endpoint authorisation names a minimum tier, not a boolean. The order is member, then admin, then owner, with `hub_admin` overriding all three.

`accounting-service` already carries most of this: `workspace_authz` (`app/app.py:178-199`) reads `workspaces` and `workspaces-owned` from the JWT, checks `hub_admin` in realm roles, and takes a `require_owner` flag. The flag is the part to replace, because a boolean cannot express "owner or admin".

The admin tier is real, not a placeholder. PR 53 on `eodhp-workspace-services` records it, and the tier reaches this service as a `workspaces-admin` JWT claim.

### How the admin claim is produced

Admin status lives in a Postgres table inside `eodhp-workspace-services`. It is not a Keycloak group, so it does not arrive in a token by itself.

Neither does workspace ownership. The `workspaces-owned` claim comes from an `oidc-api-claims-protocol-mapper` in the Keycloak realm, which calls the workspace service's own REST API when a token is issued and plucks one field from each object in the response:

```yaml
claim.name: workspaces-owned
apiUrl: http://workspace-services.workspaces:80/api/workspaces?owned
objectFieldPath: name
```

The same pattern produces `workspaces` and `billing-accounts`, so a fourth claim follows a path used three times already. The admin claim needs a client scope in `eodhp-argocd-deployment/apps/keycloak/base/realms.yaml` and an entry in the oauth2-proxy scope list (`apps/oauth2-proxy/base/values.yaml:6`).

It also needs an endpoint that does not exist in PR 53. The PR adds `GET /workspaces/{workspace-id}/admins`, which answers "who administers this workspace" and returns usernames. A claim describes the user, and the token is minted before any workspace is known, so the mapper needs the inverse: "which workspaces does this user administer", returning workspace names. The mapper takes a single URL and one field path, with no way to loop over workspaces or filter by caller, so `GET /api/workspaces?admin` mirroring the existing `?owned` filter is the shape required. This has been raised on the PR.

That endpoint should mirror `isUserWorkspaceAuthorized`'s own semantics and include workspaces the caller owns, because PR 53 treats the account owner as an implicit admin. Tier ordering means an owner would pass an admin-tier check anyway, but keeping the claim's meaning identical to the Go service's avoids a gap that would appear only for account owners.

Because the mapper reads the API at token issue, revoking admin takes effect on the next token refresh. A cached copy fed over Pulsar, which is how this service receives its other cross-service data, would hold stale authorisation for as long as the message lagged. Stale authorisation is a worse failure than stale pricing, so the claim is the right mechanism here even though it breaks the pattern used elsewhere in the service.

**Whether the credit endpoints admit the admin tier is an open decision.** PR 53 gives admins linked-account management, which is billing-adjacent and argues for letting them configure budgets. Nothing in the credit design depends on the answer: each endpoint names a minimum tier, so the change is one constant per endpoint.

Reads of the pricing policy require a valid token and no workspace membership check, since the policy is platform-wide. `/accounting/skus` and `/accounting/prices` take no token at all today (`app/app.py:429`, `:461`), so the pound-denominated values derived from the policy stay publicly reachable unless those endpoints are tightened too.

## 12. Usage reads show net charges only

The usage and ledger read endpoints return net credits: a reversal reduces the figure rather than appearing as its own row.

This needs no filtering logic. The read is `SUM(credits)` grouped by the dimension requested — period, user, or SKU — so a reversal nets against the charge it reverses automatically. Two details follow. A fully reversed charge nets to zero and is hidden rather than shown as a zero row. And individual reversal rows never appear in these endpoints, which makes the audit surface (task 9) the only place a correction is visible as an event in its own right.

## Changes to the task list

The sizings in [Credits ledger scoping](Credits%20ledger%20scoping.md) predate these decisions and predate two findings in the current code. The affected rows:

| Task | Was | Now | Reason |
|---|---|---|---|
| 1 — usage to credits with category multiplier | L | M | The cross-repo integration already exists. `accounting-service` consumes `workspace-settings` (`ingester/__main__.py:37`) and keeps a `WorkspaceAccount` cache table (`models.py:45`). Adding category is one message field, one column, and a fix to `record_mapping`. |
| 6 — periodic storage charging | L, size unclear | M | Periodic windowing already exists. `ConsumptionSampleRateIngesterMessager` converts rate samples into `BillingEvent`s in one-hour windows (`ingester/messager.py:83-140`). Storage is rate-shaped, so this extends an existing mechanism. The earlier claim that no scheduler exists was wrong. |
| 0, 3, 7, 8 — policy version items | M, S, M, S | One mechanism | Decision 3 merges these into the versioned policy bundle and operations on its identifier. |
| 11 — budget and threshold model | L, size unclear | M | Decision 4 settles the enforcement question that made the size unclear. The model is per type, but a breach only publishes a message. |

Two items change status rather than size:

- **Alerting is no longer a gap; it is the enforcement mechanism.** Decision 4 makes the breach message the entire control, so publishing it is a core task rather than a nice-to-have. Its consumer remains out of scope per decision 9.
- **Historical re-pricing is a new task, sized M.** See decision 8.

One caveat on the message-paced windowing behind task 6: generation runs up to the start of the hour containing each arriving message, so it stalls when samples stop arriving. It is not clock-driven.

## Defects found in existing code

Neither of these is caused by this work, and both affect it.

**Unmetered final hour after resource deletion.** When a resource is deleted, part of its last hour goes uncharged. Collectors are supposed to emit a zero-rate message at deletion and an hour later; none do. Recorded in a code comment at `accounting_service/ingester/messager.py:100-104`, which notes "deletion is currently rare."

**The `workspace-settings` schema has already drifted between producer and consumer.** The Python record declares `member_group` and `last_update` (`eodhp-utils/eodhp_utils/pulsar/messages.py:151-158`); the Go producer sends `Owner` and `LastUpdated` (`eodhp-workspace-manager/models/workspace_settings.go`). This does not bite today only because `accounting-service` reads `name` and `account` alone. Decision 6 adds a field to this contract, so the Go and Python sides must change together, and the existing drift is worth fixing in the same pass.

A third item was not a defect but a gap: the config loader was a bare `yaml.safe_load` into a dict with no schema validation. Decision 3 gives the loader more to do, which is the point at which validation earned its place. Closed by T2 — see *The configuration document* in the [schema note](Credits%20ledger%20schema.md).

## Still open

- The storage-billing decision referenced by task 6. Decision 4 and the windowing finding reduce its scope, but the charging cycle and proration rules are still undecided.
- Reconciliation and backfill for missed billing events. Deduplication on event UUID prevents double-charging, but nothing detects gaps from ingester downtime, consumer lag, or a dropped message. The unmetered-deletion defect above is one instance of the same class.
- Whether reading a balance is available to every member of a workspace or to the workspace admin alone. Decision 1 assumes members can read; this is not confirmed.
- Which credit endpoints admit the workspace admin tier now that PR 53 makes it real. Budget configuration is the likeliest candidate. See D11.
- How often a calibration pass is expected to run. This affects how much tooling the policy-version mechanism deserves, not whether it is correct.
