# Discover Logs

## Purpose

This operation shows how to create a simple log discovery to quickly collate and view logs in Kibana from different parts of the platform.

## When to Use

Use this operation when you want to create a view into specific logs, perhaps combined from multiple different sources within the platform.

## Operation

1. Visit https://logs.eodatahub.org.uk
2. Log in by following [Access Kibana Logs](./access-kibana-logs.md)
3. Select Analytics app
4. Select Discover app
5. You are now presented with all of the logs from the platform (no filters applied). To make sense of these we will apply filters for what we want to view. For this example, we will create a view into the authentication layer of the platform, which are the logs from the auth agent and oauth2-proxy instances.
6. You will need some details of the Kubernetes pod deployments for these services. You can either get these from `kubectl` or from ArgoCD.

   **ArgoCD**

   1. [Log into ArgoCD](../argocd/web-ui.md)
   2. Select ArgoCD app you wish to view pod info for
   3. On desired resource (usually pod), click ... (or click on icon) and Details
   4. Select Live manifest tab

   **kubectl**

   1. `kubectl get -n <namespace> pods` to find the available pods
   2. `kubectl get -n <namespace> pod/<pod-name> -o yaml` to see the manifest

7. Inspect the manifest for queriable data. Usually container name (`spec.containers.*.name`) is suitable, assuming it is unique to the pod containers you wish to view logs for. If it is not unique, you will need to find additional queriable fields you can use, such as `metadata.labels.app`. Note the fields you wish to query.
8. Back in Kibana, start building our filters. We will build filters for containers named either `auth-agent` or `oauth2-proxy` (this includes both `oauth2-proxy-platform` and `oauth2-proxy-workspaces` pods)
   1. Add filter (top left, blue '+')
   2. Select a field, the filed we want is `kubernetes.container.name`. Note that the fields here do not map directly to the pod specs, some searching and trial and error may be required to find the right field.
   3. Select operator 'is'
   4. Select a value, enter `auth-agent`.
   5. Add an OR condition using buttons to the right
   6. Repeat process for `oauth2-proxy`
   7. Add filter
9. Now we want to define the columns from these logs that we are interested in. Use the left side bar to filter and select the following, clicking the '+' to add the columns to the data view.
   1. `kubernetes.container.name`
   2. `message`
   3. `error.message`
10. At this point we can save our view for re-use. Give it a title, "Auth Logs" in this case. Add a meaningful description and any desired tags, e.g. `auth`.
11. You can now revisit this view from the Discovery app using the Open (folder) icon from the top right.

## Requirements

- ArgoCD UI or `kubectl` access to inspect pod manifests
- Access to Kibana UI

## Useful Information

- The ELK stack is very powerful, this is just a simple example to demonstrate how to get started increasing the observability of the platform.
