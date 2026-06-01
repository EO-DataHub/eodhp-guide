---
title: JupyterHub Resilience and Resource Limits
doc_status: unreviewed
tags:
  - jupyter
  - notebooks
last_reviewed:
reviewed_by:
review_notes:
---

# JupyterHub Resilience and Resource Limits

Covers the resource limit configuration that prevents notebook pods from starving other workloads and how to prepare for high-traffic events.

**Reference:** [JupyterHub service](../../reference/services/jupyter.md) · **See also:** [Dask Integration](dask-integration.md)

---

## Resource limits

Without resource limits, a single notebook pod can consume all available CPU and memory on its node, degrading or killing other pods on that node. Resource limits are configured in the JupyterHub spawner profile in [eodhp-argocd-deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment).

### Default profile

| | Request | Limit |
|-|---------|-------|
| CPU | 1 | 2 |
| Memory | 2 GB | 4 GB |

The request guarantees the resource on the node; the limit caps what the pod can consume. The gap between request and limit allows for overprovisioning when notebooks are idle (idle usage is typically <5% of the 4 GB memory limit).

### Observed usage

- Light workloads (data discovery, metadata queries): memory well under 1 GB
- Heavier workloads (visualisation, raster processing): memory peaks around 1 GB

The 4 GB limit is adequate for typical workshop use cases and could be reduced if node capacity becomes a concern.

### Adjusting limits for specific users or events

Limits are set per spawner profile in the JupyterHub Helm values in `eodhp-argocd-deployment`. To increase limits for a specific group:

1. Edit the relevant spawner profile in the Helm values.
2. Test the change on dev.
3. Raise a PR to deploy to staging, verify, then promote to production.
4. After the event or testing period, revert if the higher limits are not needed long-term.

**Note:** When a `ResourceQuota` is active on a namespace (e.g. workspaces with Dask enabled), all pods in that namespace — including the JupyterHub single-user pod — must declare resource requests and limits. If the spawner profile does not set these, new server launches will be rejected by the quota. See [Dask Integration](dask-integration.md) for details.

---

## Preparing for high-traffic events

For events where many users will be running notebooks simultaneously (workshops, conferences):

1. **Confirm which notebooks will be used** — review memory and CPU requirements in advance.
2. **Test with resource limits on dev** — launch a server, run the workshop notebooks, check pod metrics via Headlamp or Grafana.
3. **Check idle shutdown behaviour** — JupyterHub is configured to cull idle servers after a timeout. Confirm the timeout is appropriate for the event format (e.g. a workshop with breaks should not cull servers mid-session).
4. **Deploy limit changes to staging, then prod** — follow the standard PR → staging → prod flow.
5. **Monitor during the event** — use Headlamp or Grafana to watch CPU/memory usage per pod. Limits prevent any single pod from causing cascading failures on others.

---

## Monitoring

Pod-level resource usage is visible in:

- **Headlamp** — available in-cluster, useful when Grafana auth is unavailable.
- **Grafana** — integrated with Prometheus; provides historical data and alerting.
- **kubectl:** `kubectl top pods -n <namespace>`

---

## Known issues

- **Namespace assignment:** JupyterHub servers for workspaces with certain name patterns can be launched into incorrectly named namespaces (e.g. `ws-x-4eitesting---e2ea0340` instead of `ws-4eitesting`). If a user's server is missing or behaving unexpectedly, verify the namespace the pod was actually scheduled into.
