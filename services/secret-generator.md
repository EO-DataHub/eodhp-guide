# Secret Generator

## Summary

Secret Generator is a Kubernetes controller which generates values in appropriately annotated Kubernetes Secrets. It's used to auto-generate secret material the cluster needs that doesn't need to be coordinated with external sources.

This is unmodified third-party software.

### Dependent Services

Many services depend on external-secrets for initial configuration - see `grep -r secret-generator.v1.mittwald.de apps` in the ArgoCD repo to find them. These include Pulsar, oauth2-proxy, auth-agent (via Redis), stac-fastapi and the web presence. Large parts of the cluster will fail to initialize without it.

Once secrets have been generated and the cluster is running these services no longer need secret-generator.

### Configuration

Secret Generatr is configured in [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, apps/secret-generator directory.

### Control

To restart service run `kubectl rollout restart -n secret-gen deploy` for Kubernetes cluster or use ArgoCD UI to restart.

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

None.
