---
title: Developer branching and PR workflow
doc_status: ok
tags:
  - gitops
last_reviewed:
reviewed_by:
review_notes: "Contributing/process section from docs/Development.md"
---

# Developer branching and PR workflow

When a developer begins work on an issue they take a branch from `main`. The branch should follow Gitflow naming and mention the Jira issue key, e.g. `feature/EODHP-123-my-new-feature`. Work on the feature branch and, when ready, create a PR and assign reviewer(s). Once the PR has been accepted, merge back into `main`.

`main` branch is always the source of truth for the latest configuration of the EODH deployment.

For how that ties into Kargo and environment branches, see [GitOps branching and Kargo promotion](../../explanation/development-and-governance/gitops-branching-and-kargo-promotion.md).
