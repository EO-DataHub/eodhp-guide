# Grafana

## Summary

Grafana is connected to Prometheus and is used for visualizing system metrics. It can be accessed at https://grafana.eodatahub.org.uk/ by any Keycloak user with the `admin` role.

Typical uses are to visualize time series of CPU and memory use of pods and to visualize time series of harvest pipeline Pulsar topics' message rates and backlogs.

Grafana is unmodified third-party software.

### Dependent Services

None

### Configuration

Grafana is configured in [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, apps/grafana directory.

### Control

To restart service run `kubectl rollout restart -n grafana deployment grafana ` for Kubernetes cluster or use ArgoCD UI to restart.

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

Prometheus and Keycloak.
