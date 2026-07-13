---
title: Shared Renovate configuration
doc_status: ok
tags:
  - ci-cd
  - renovate
  - dependencies
last_reviewed: 2026-06-29
reviewed_by: recmanj
review_notes:
---

# Shared Renovate configuration

[Renovate](https://docs.renovatebot.com/) is an automated dependency-update bot. It scans a repository's manifests and lock files, then raises pull requests to update outdated dependencies — keeping packages, Docker images, and GitHub Actions current with minimal manual effort.

`renovate.json5` in the [`github-actions`](https://github.com/EO-DataHub/github-actions) repository is a **shareable [Renovate](https://docs.renovatebot.com/) preset** that standardises dependency-update behaviour across EO Data Hub repositories. For the reusable CI workflows in the same repository, see [Reusable CI workflows](reusable-workflows.md).

## Adopting the preset

A repository opts in by extending the preset from its own `.github/renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>EO-DataHub/github-actions//renovate.json5"]
}
```

## What the preset configures

**Core behaviour**

- Dependency Dashboard issue, semantic commit messages, and automatic config migration.
- Ignores modules and test directories (`:ignoreModulesAndTests`).

**Grouping**

- Monorepo and recommended grouping presets.
- One batched PR per ecosystem per week for non-major updates: GitHub Actions, npm, Python, and Docker images/digests.

**Scheduling**

- Updates run on a weekly schedule; lock-file maintenance runs weekly and is automerged.

**Automerge**

- All non-major updates (`minor`, `patch`, `pin`, `digest`) automerge once CI passes, after a `minimumReleaseAge` cooldown of **7 days**.
- **Major** updates are raised as individual, non-automerged PRs for manual review.

**Pinning**

- Dev dependencies, Docker image digests, and GitHub Action digests are pinned.

**Security**

- OpenSSF Scorecard and merge-confidence age badges.
- `vulnerabilityAlerts` enabled, labelled `security`, scheduled "at any time" (i.e. outside the weekly window).

**Other**

- `prConcurrentLimit` of 5; all PRs labelled `dependencies`.
- Internal `EO-DataHub/github-actions` reusable workflows are excluded from updates (consuming repos pin them to `@main`).
- Python dev `dependency-groups` (PEP 735) are pinned.
