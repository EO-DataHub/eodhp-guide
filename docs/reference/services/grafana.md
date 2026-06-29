---
title: Grafana
doc_status: ok
tags:
  - grafana
  - observability
last_reviewed: 2026-06-26
reviewed_by: recmanj
review_notes: Updated to reflect VictoriaLogs datasource and Logs Explorer dashboard.
---
# Grafana

## Summary

Grafana is the platform's primary observability UI. It can be accessed at https://grafana.eodatahub.org.uk/ by any Keycloak user with the `admin` role.

Grafana connects to two data sources:

| Datasource | Type | Used for |
|-----------|------|----------|
| Prometheus | Metrics | CPU/memory usage, Pulsar message rates, service health |
| VictoriaLogs | Logs | Kubernetes container logs from all platform pods |

Typical uses include visualising time series of CPU and memory use of pods, monitoring Pulsar harvest pipeline message rates and backlogs, and searching platform logs via the **EODH Platform Logs Explorer** dashboard.

Grafana is unmodified third-party software.

### Dependent Services

None

### Configuration

Grafana is configured in the [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment), `apps/grafana` directory.

### Control

```sh
kubectl rollout restart -n grafana deployment grafana
```

Or use the ArgoCD UI to restart. To stop the service, remove it from ArgoCD configuration.

### Dependencies

Prometheus, VictoriaLogs, and Keycloak.

**Related:** [Monitor resources](../../how-to/observability-logging/monitor-resources.md), [Add a custom Grafana dashboard](../../how-to/observability-logging/add-a-custom-dashboard.md), [Access platform logs](../../how-to/observability-logging/access-logs.md), [VictoriaLogs](victorialogs.md).
