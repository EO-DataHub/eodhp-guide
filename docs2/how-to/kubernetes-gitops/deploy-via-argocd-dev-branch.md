---
title: Deploy via Argo CD dev branch
doc_status: ok
last_reviewed: 2026-05-20
reviewed_by: geodowd
review_notes: "From docs/Development.md (Deploying to a Development Cluster)"
---

# Deploy via Argo CD dev branch

To deploy new features to a development cluster for testing you need to commit your changes to the Argo CD branch for that cluster. These changes will typically be updates to the Kubernetes configuration itself or updating a pod image to a newer version.

Once the changes are committed and pushed to the Git remote, Argo CD will automatically deploy your changes to the cluster. Argo CD polls for updates to the Git remote every 3 minutes but you can speed this up by refreshing the app in the Argo CD UI. Alternately, [webhooks](https://argo-cd.readthedocs.io/en/stable/operator-manual/webhook/) can be configured between Argo CD and the Git repo to trigger updates on any commits to the Git remote.

**Context:** [GitOps branching and Kargo promotion](../../explanation/development-and-governance/gitops-branching-and-kargo-promotion.md).
