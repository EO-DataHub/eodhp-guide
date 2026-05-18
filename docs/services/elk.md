# ELK / Elastic Stack

## Summary

The Elastic stack collects, stores and presents logs from around the system. It can be accessed at https://logs.eodatahub.org.uk/ but is not integrated with Keycloak. The `elastic` user can log in using the password found with `kubectl get secret -n elk elasticsearch-es-elastic-user -o yaml` (you will need to base64-decode it). Kubernetes logs are ingested into Elastic and can be searched and browsed with this UI.

Some services, particularly those in the harvest pipeline and which are based on the eodhp-utils library, submit structured logs including data such as the workspace involved (`json.workspace` field in the UI). This allows a single action to be traced across multiple services with `json.otelTraceID`.

This is unmodified third-party software.

### Dependent Services

None

### Configuration

The Elastic Stack is configured in [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, apps/elk directory.

### Control

To restart service run `kubectl rollout restart -n elk sts` and `kubectl rollout restart -n elk deploy` for Kubernetes cluster or use ArgoCD UI to restart.

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

None
