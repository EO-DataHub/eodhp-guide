---
title: Discover Logs
doc_status: remove-candidate
last_reviewed:
reviewed_by:
review_notes: Migrated from docs/operations/observability/discover-logs.md.
---
# Discover Logs

## Purpose

This operation shows how to create a simple log discovery to quickly collate and view logs in Kibana from different parts of the platform.

## When to Use

Use this operation when you want to create a view into specific logs — perhaps combined from multiple sources within the platform.

## Operation

1. Visit https://logs.eodatahub.org.uk
2. Log in by following [Access Kibana Logs](access-kibana-logs.md).
3. Select **Analytics** app.
4. Select **Discover** app.
5. You are now presented with all of the logs from the platform (no filters applied). To make sense of these, apply filters for what you want to view. For this example we create a view into the authentication layer: logs from the auth agent and oauth2-proxy instances.
6. You need some details of the Kubernetes pod deployments for these services. You can either get these from `kubectl` or from Argo CD.

   **Argo CD**

   1. [Open the Argo CD web UI](../kubernetes-gitops/argocd-web-ui.md).
   2. Select the Argo CD app you wish to view pod info for.
   3. On the desired resource (usually a pod), open the menu (**…**) or icon and choose **Details**.
   4. Select the **Live manifest** tab.

   **kubectl**

   1. `kubectl get -n <namespace> pods` — list pods
   2. `kubectl get -n <namespace> pod/<pod-name> -o yaml` — show the manifest

7. Inspect the manifest for queriable data. Usually the container name (`spec.containers.*.name`) is suitable, if it uniquely identifies the pod containers you care about. If it is not unique, find additional fields such as `metadata.labels.app`. Note the fields you plan to query.
8. Back in Kibana, build filters — for containers named either `auth-agent` or `oauth2-proxy` (this includes both `oauth2-proxy-platform` and `oauth2-proxy-workspaces` pods):

   1. **Add filter** (top left, blue **+**).
   2. Select field `kubernetes.container.name`. (Fields here may not map 1:1 to pod specs; some trial and error may be needed.)
   3. Operator: **is**.
   4. Value: `auth-agent`.
   5. Add an **OR** condition using the controls to the right.
   6. Repeat for `oauth2-proxy`.
   7. Confirm the filters.

9. Define columns from these logs via the left sidebar — filter and select the following and use **+** to add columns:

   1. `kubernetes.container.name`
   2. `message`
   3. `error.message`

10. Save the view for reuse. Example title: **Auth Logs**. Add a meaningful description and tags, for example `auth`.
11. Reopen saved views from Discover using **Open** (folder icon) at the top right.

## Requirements

- Argo CD UI or `kubectl` access to inspect pod manifests
- Access to the Kibana UI

## Useful Information

The ELK stack is very powerful; this is a simple example to get started improving observability of the platform.

**Related:** [Access Kibana Logs](access-kibana-logs.md).
