---
title: Monitoring Resource Usage Using Grafana
doc_status: ok
last_reviewed:
reviewed_by:
review_notes: Migrated from docs/operations/observability/monitor-resources.md.
---
# Monitoring Resource Usage Using Grafana

## Purpose

This operation allows a Hub Admin to monitor specific resource usage down to namespace (workspace) and pod level. This can help pinpoint namespaces or pods which are using excessive amounts of RAM or CPU to identify the causing operations.

## When to Use

If pods are starting to fail in workspaces — for example during Notebook processing or Workflow execution — then an Admin may wish to bugfix this issue. Grafana can help determine if there are any issues with resource usage within the cluster. If the issue might be related to resource usage, Grafana can help identify which pods (or containers) are using excessive resources.

Grafana can also be used to monitor the Pulsar messaging service, in particular the rate at which catalogue harvester messages are propagating through the harvest pipeline. This can help identify backlogs or bottlenecks in one of the steps.

## Operation

To access the Grafana UI, a user with an Admin account (they have the `hub-admin` role assigned) can visit the Grafana domain at https://grafana.eodatahub.org.uk/ and authenticate with their Admin Hub account. To view Kubernetes resource usage across namespaces, navigate to the **Dashboards** page in the left pane, then select **Kubernetes** > **Kubernetes / Views / Namespaces**. This opens the namespaces dashboard. To view data across all namespaces, use the drop-down at the top and select **All**. The dashboards on this page can be used to view CPU and RAM usage within the Kubernetes cluster. The **Overview** section shows live CPU and RAM usage across all pods in the selected namespace(s). Using the **Resources** section, you can monitor resource usage by individual pods — select pod names to isolate them on the graphs. To view details for a specific namespace, choose the workspace from the drop-down at the top. Select **Refresh** to reload the dashboard. Other options are available for finer analysis, such as time filtering and resolution.

To view Pulsar backlog information instead, navigate to **Dashboards** and select **Pulsar Backlog**. That page provides graphs displaying counts of message acknowledgements and backlog size for specific topics, among other metrics.

When viewing pod resources, note that Notebook pods are prefixed with `jupyter-` and workflow pods are prefixed with the name of the step being run, for example `water-quality-`. You can use the Kubernetes CLI to view more pod details with `kubectl -n <namespace> get pods <pod-name>`.

## Requirements

- A Hub account with the `hub-admin` role
- Access to https://grafana.eodatahub.org.uk/

## Useful Information

- You can view more detailed resource usage for individual containers inside pods via **Dashboards** > **Kubernetes** > **Kubernetes / Views / Pods**.
- If you identify pods using excessive resources, you can view logs with `kubectl -n <namespace> logs <pod-name>`. You may wish to terminate those pods or update requests and limits in the pod manifest, in particular for workflow steps.
- To find a pod when you do not know the namespace: `kubectl get pods -A | grep <pod-name>`.
- Consider the node type deployed in the cluster: it constrains CPU and RAM available on each cluster node.
- If dashboard panels appear blank, adjust the timescale at the top right (for example increase from 15 minutes to 30 minutes).
- Data shown in these Grafana dashboards feeds billing and accounting information.

**Related:** [Add a custom Grafana dashboard](add-a-custom-dashboard.md).
