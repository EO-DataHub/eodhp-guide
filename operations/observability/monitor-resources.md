# Monitoring Resource Usage Using Grafana

## Purpose

This operation allows a Hub Admin to monitor specific resource usage down to namespace (workspace) and pod level. This can help pinpoint namespaces or pods which are using excessive amounts of RAM or CPU to identify the causing operations.

## When to Use

If pods are starting to fail in workspaces, for example during Notebook processing or Workflow execution, then an Admin may wish to bugfix this issue. Grafana can help a user to determine if there are any issues with resource usage within the cluster. If the issue might be related to resource usage, then Grafana can help identify which pods, or even containers, are using excessive resources.

## Operation

To access the Grafana UI, a user with an Admin account, meaning they have the `hub-admin` role assigned to their account, can visit the Grafana domain at https://grafana.eodatahub.org.uk/ and authenticate with their Admin Hub account. To view Kubernetes logs across namespaces, navigate to the `Dashboards` page in the left pane, then select `Kubernetes` >  `Kubernetes / Views / Namespaces`. This will open the namespaces dashboard. To view data across all namespaces, use the drop-down field at the top and select `All`. The various dashboards on this page can be used to view CPU and RAM usage within the Kubernetes cluster. The Overview section shows live CPU and RAM usage across all pods in the selected namespace(s). Using the Resources section, you can further monitor resource usage by individual pods, selecting pod names to isolate them on the provided graphs. To view details for a specific namespace, again select the drop-down at the top of the page and select the workspace you wish to interrogate. Hit Refresh to reload the dashboard with your selected configuration. Other options are also available for more precise analysis, such as time filtering and resolution.

When viewing pod resources, note that any Notebook pods will be prefixed with `jupyter-` and any workflow pods will be prefixed with the name of the step being run, for example `water-quality-`. You can then use the Kubernetes CLI to view more pod details using `kubectl -n <namespace> get pods <pod-name>`

## Requirements

- A Hub account with the `hub-admin` role
- Access to https://grafana.eodatahub.org.uk/

## Useful Information

- You can also view more detailed resource usage data for individual containers within pods using the `Dashboards` > `Kubernetes` > `Kubernetes / Views / Pods` pages. 
- If you identify any pods that you think are using excessive resources that are causing issues, you can then view the logs for that pod, using the Kubernetes CLI, `kubectl -n <namespace> logs <pods-name>`. You may wish to terminate any pods that are using excessive resources, or you might need to update the request and limits defined in the pod manifest, in particular for workflow steps.
- If you wish to find a given pod using the Kubernetes CLI, you can use the following command, if you are not sure of the namespace: `kubectl get pods -A | grep <pod-name>`. This will return all pods running in the cluster that contain the `pod-name` in their name.
- It is important to consider the Node type deployed to the cluster, as this will define the restrictions that apply to CPU and RAM available throughout each cluster node.
- If any dashboard fields appears blank, ensure you have selected an appropriate timescale in the top right, for example if no data is seen for the last 15 minutes, consider increasing it to 30 minutes.
- Note, the data that is displayed in these Grafana dashboards provides the basis for billing and accounting information