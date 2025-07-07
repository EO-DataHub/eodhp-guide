# Argo Workflows and Argo Events.

## Summary

Argo Events is used to trigger catalogue harvesters, either on a schedule or (for Git-based harvesters) in response to webhook calls from GitHub. Argo Workflows runs a `harvester-test` workflow which tests the harvest pipeline functionality and includes a UI to Argo Events (which is not currently used but may be in the future).

Both may play a larger role in the future, especially for user-managed harvesting and for future notification-driven system behaviour.

### Dependent Services

Airbus harvesting depends on Argo Events to trigger it. Harvesting of catalogue and workflow configuration from Git also depends on it, including setting up the initial 'public', 'commercial' and 'user' Catalogs.

### Configuration

They are configured in the [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, apps/argo-workflows and argo-events directories.

### Control

To restart service run `kubectl rollout restart -n argo-workflows deploy` and `kubectl rollout restart -n argo-events deploy` for Kubernetes cluster or use ArgoCD UI to restart.

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

None
