---
title: Grafana
doc_status: ok
tags:
  - grafana
  - observability
last_reviewed:
reviewed_by:
review_notes: Migrated from docs/services/grafana.md.
---
# Grafana

## Summary

Grafana is connected to Prometheus and is used for visualizing system metrics. It can be accessed at https://grafana.eodatahub.org.uk/ by any Keycloak user with the `admin` role.

Typical uses are to visualize time series of CPU and memory use of pods and to visualize time series of harvest pipeline Pulsar topics' message rates and backlogs.

Grafana is unmodified third-party software.

### Dependent Services

None

### Configuration

Grafana is configured in the [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment), `apps/grafana` directory.

### Control

To restart service run `kubectl rollout restart -n grafana deployment grafana ` for Kubernetes cluster or use ArgoCD UI to restart.

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

Prometheus and Keycloak.

**Related:** [Monitor resources](../../how-to/observability-logging/monitor-resources.md), [Add a custom Grafana dashboard](../../how-to/observability-logging/add-a-custom-dashboard.md).
