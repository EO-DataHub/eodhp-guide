---
title: TiTiler Resilience and Scaling
doc_status: unreviewed
tags:
  - titiler
  - kubernetes
last_reviewed:
reviewed_by:
review_notes:
---

# TiTiler Resilience and Scaling

Covers TiTiler's resource configuration, load testing findings, event-day scaling procedure, and the path towards horizontal pod autoscaling.

**See also:** [Private Data Rendering](private-data-rendering.md) — the distinct roles of the two TiTiler deployments.

---

## Resource configuration

There are two TiTiler deployments in the cluster:

| Deployment | Replicas | CPU request | CPU limit | Memory request | Memory limit |
|------------|----------|-------------|-----------|---------------|--------------|
| `titiler` | 2 | 6 | 8 | 28 Gi | 32 Gi |
| `titiler-stacapi` | 3 | 6 | 8 | 28 Gi | 32 Gi |

Each pod's resource requests are large enough that only **one pod fits per EC2 node**, so replica count directly maps to dedicated EC2 instances. There is no Horizontal Pod Autoscaler (HPA) in place.

TiTiler runs at well under 5% average CPU utilisation during normal operation — the replica counts are set for peak-event headroom rather than mean usage. The replica counts are the primary scaling lever; changing per-pod resource limits alone is insufficient because the node-per-pod sizing would need to be revisited at the same time.

---

## Event-day scaling procedure

Prior to events with high concurrent use (workshops, demos, hackathons), increase the replica count manually:

1. Update the replica count in the relevant values in `eodhp-argocd-deployment` and promote through Kargo, or patch the deployment directly for an urgent event.
2. Monitor CPU and memory during the event (see [Monitoring](#monitoring)).
3. Revert to the baseline replica count after the event.

A 5× increase in replicas has been used for large workshop events and found to be adequate.

---

## Performance tuning: `WEB_CONCURRENCY`

The default Uvicorn worker count is the main bottleneck under concurrent tile requests. The `WEB_CONCURRENCY` environment variable controls how many worker processes TiTiler runs per pod.

| `WEB_CONCURRENCY` | Throughput (100-user test) | Median response time |
|-------------------|---------------------------|---------------------|
| 1 (default) | ~10 req/s | ~9 s |
| 4 | ~35 req/s | ~3 s |
| 8 (current) | ~35–40 req/s (plateau) | ~2–3 s |

`WEB_CONCURRENCY=8` is the current production baseline.

**Rule of thumb:** Set `WEB_CONCURRENCY` to approximately the number of vCPUs on the pod. With 8 vCPUs allocated per pod, 8 workers is appropriate.

---

## Load testing with Locust

A Locust harness has been developed for load testing the COG tiles endpoint. The test:

1. Queries the STAC API for a set of Sentinel-2 ARD items with cloud cover ≤ 10% over a UK bounding box.
2. Fetches geographic bounds for each COG via TiTiler's `/cog/bounds` endpoint.
3. Randomly requests PNG tiles across zoom levels 8, 10, 12, 14.
4. Also simulates map panning by requesting 3×3 tile grids.

```bash
export EODH_API_TOKEN=<your-api-token>
export TITILER_HOST=https://eodatahub.org.uk/titiler/core

locust -f locustfile.py \
  --headless \
  --users 100 \
  --spawn-rate 10 \
  --run-time 10m \
  --html=cog-report-100-10-10.html
```

### Key findings

**COG tiles (Sentinel-2 ARD, hosted on CEDA):**

- At 50 concurrent users, TiTiler handled load well with minimal errors.
- At 100–200 users, errors increased significantly — mostly `500` responses attributable to CEDA throttling the upstream COG source rather than TiTiler itself being resource-constrained.

**Implication:** For data hosted directly in EODH workspace S3 (purchased commercial data, user COGs), performance should scale considerably better. CEDA throttling is an external limit, not a TiTiler architectural constraint.

**Xarray/NetCDF tiles:**

Response times are poor even at low concurrency, likely due to how the underlying NetCDF files are chunked rather than TiTiler itself. Most near-term usage is expected to be COG data, so this has not been investigated further.

---

## Future work: Horizontal Pod Autoscaling

The current fixed replica count requires manual scaling for events. An HPA policy scaling on CPU or memory utilisation would handle load spikes automatically and reduce cost during quiet periods.

Architectural questions to address before HPA work is finalised:

- **Why is there a dedicated node group for TiTiler pods?** If TiTiler shared a node group with other workloads, pod-per-node sizing would no longer be required and scaling would be cheaper.
- **Why are resource requests sized so that only one pod fits per node?** This isolation choice needs revisiting for cost-efficient autoscaling.
- **What are the response-time SLOs for tile requests?** Autoscaler tuning (target CPU %, scale-up/down delays) cannot be sensibly set without a target latency.

---

## Monitoring

Grafana dashboards for TiTiler pods:

```
https://grafana.eodatahub.org.uk/d/k8s_views_pods/kubernetes-views-pods?var-namespace=titiler
```

Key metrics to watch during an event: CPU utilisation, memory utilisation, request rate, and error rate per pod. A sustained CPU spike close to the limit is the clearest signal that replica count should be increased.
