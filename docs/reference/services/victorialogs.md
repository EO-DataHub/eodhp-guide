---
title: VictoriaLogs
doc_status: ok
tags:
  - victorialogs
  - observability
last_reviewed: 2026-06-26
reviewed_by: recmanj
review_notes: New reference page; replaces elk.md.
---
# VictoriaLogs

## Summary

VictoriaLogs collects and stores Kubernetes container logs from across the platform. Logs are collected by a Vector DaemonSet running on every node and forwarded to two VictoriaLogs instances (`victorialogs-a` and `victorialogs-b`) for high availability. A Grafana datasource and dashboard expose the logs to operators.

- **Retention:** 7 days
- **Storage:** 20 GiB per instance (persistent block storage)
- **Access:** via Grafana — **Dashboards > VictoriaLogs > EODH Platform Logs Explorer**
- **Query language:** [LogsQL](https://docs.victoriametrics.com/victorialogs/logsql/)

### Deployment Model

The platform runs across [two AWS availability zones](../../architecture/cloud-architecture.md), and VictoriaLogs mirrors that: **two independent single-node instances, one per AZ.** `victorialogs-a` and `victorialogs-b` are deployed in the `victorialogs` namespace from the `victoria-logs-single` Helm chart (single-node, **not** cluster mode) as two separate Helm releases sharing the same `values.yaml`. Each runs as its own StatefulSet with a dedicated 20 GiB persistent volume and 7-day retention. A topology-spread constraint (`topology.kubernetes.io/zone`, `whenUnsatisfiable: DoNotSchedule`) pins the two instances to different zones, so a complete copy of the logs lives in each AZ.

High availability comes from duplicating ingestion rather than clustering:

- **Writes (active/active):** Vector defines two sinks — `victorialogs_a` and `victorialogs_b` — both fed from the same enrichment pipeline, so every log record is written independently to *both* instances.
- **Reads:** A shared `victorialogs-query` Service fronts both instances (selecting `app.kubernetes.io/name: victoria-logs-single` with `trafficDistribution: PreferClose`). The Grafana datasource queries this service, which routes to whichever instance is healthy and closest.

If an entire availability zone is lost, the surviving instance in the other AZ still holds a complete copy of the logs, so ingestion and queries continue with no data loss. A PodDisruptionBudget (`maxUnavailable: 1`) preserves this during voluntary disruptions.

### Log Fields

Vector enriches log records before forwarding them. Fields available in queries:

| Field | Source |
|-------|--------|
| `kubernetes.pod_name` | Pod name |
| `kubernetes.pod_namespace` | Kubernetes namespace |
| `kubernetes.container_name` | Container name |
| `kubernetes.app_name` | `app.kubernetes.io/name`, `app`, or `k8s-app` label |
| `kubernetes.component` | `app.kubernetes.io/component` label |
| `kubernetes.instance` | `app.kubernetes.io/instance` label |
| `level` | Extracted from JSON log fields (`level`, `severity`, `levelname`) or inferred from message text |
| `stream` | `stdout` or `stderr` |
| `_msg` | Log message body |

Structured logs (JSON) are unpacked: the `message` or `msg` field becomes `_msg`; remaining fields are stored under `log.*`.

### Dependent Services

Grafana (datasource and dashboard), Vector (log shipping DaemonSet).

### Configuration

Configured in the [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment), `apps/victorialogs` directory.

The Vector configuration is in `apps/victorialogs/base/vector-values.yaml`. VictoriaLogs Helm chart configuration is in `apps/victorialogs/base/values.yaml`.

### Control

```sh
# Restart VictoriaLogs instances
kubectl rollout restart -n victorialogs statefulset

# Restart Vector (log shipper)
kubectl rollout restart -n victorialogs daemonset/vector

# Or use the ArgoCD UI to sync/restart
```

To stop the service, remove the application from the ArgoCD configuration.

### Dependencies

Kubernetes node access for Vector log collection; block-storage class for persistent volumes.

**Related:** [Access platform logs](../../how-to/observability-logging/access-logs.md), [Search and filter logs](../../how-to/observability-logging/discover-logs.md), [Grafana](grafana.md).
