# Restart a Hub Service using ArgoCD

## Purpose

Sometimes hub services may be required to be restarted manually. This procedure describes how this can be achieved through the ArgoCD web UI.

## When to Use

Use this procedure when a manual restart of a hub service is required, for instance if the service has become non-responsive, or some part of the services configuration has changed (e.g. AWS Secret) and the service was not automatically restarted.

## Operation

This procedure assumes you already havev access to the ArgoCD web UI.

1. From the /applications page in the ArgoCD web UI, click into the ArgoCD app containing the service you wish to restart.
2. Find the service to restart in the dependency tree
3. Click the three dots on the service card and click restart. Alternately, if it is only an individual pod you wish to restart, you can click on the pod's three dots and delete that pod. It will automatically be recreated by ArgoCD.

## Requirements

- Access to ArgoCD web UI

## Useful Information

- Care should be taken when restarting platform services. It is recommended only thos sufficiently familiar with the platform operation carry out this procedure.
