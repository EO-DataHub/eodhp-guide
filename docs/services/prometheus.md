# Prometheus

## Summary

Prometheus collects system metrics into a database. Prometheus is unmodified third-party software, please see Prometheus's own documentation for details.

### Dependent Services

- Grafana uses Prometheus as the source (and storage point) of the data it displays.
- The accounting system's Compute Collector depends on Prometheus to collect workspace CPU and memory consumption data. In the event of Prometheus data loss, the accounting system will be unable to bill for compute consumption between the Compute Collectors last fetch from Prometheus and the time Prometheus service is restored.

### Configuration

Prometheus is configured in [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, apps/prometheus directory.

Multiple other services apply Prometheus annotations to their Kubernetes resources in order to configure Prometheus to collect metrics from them.

### Control

To restart service run `kubectl rollout restart -n prometheus deploy` for Kubernetes cluster or use ArgoCD UI to restart.

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

Prometheus will not fail if individual services' Prometheus metrics endpoints fail but will stop collecting data. Services with endpoints are the core Kubernetes system components, Pulsar, nginx and cert-manager.
