# Replicator

## Summary

Replicator is a Kubernetes controller which copies appropriately annotated Kubernetes resources between namespaces. This is used to copy certificates from the `certs` namespace into the namespaces of the services which must access them.

This is unmodified third-party software.

### Dependent Services

linkerd will be unable to rotate its identity (every few days) or root certificates (every 1 (existing certifciate) or 10 (new certificates) years) if this is not available. This will break the entire cluster.

nginx will also be unable to access new TLS certificates from Let's Encrypt when they expire. This will make the cluster inaccessible to users.

Certificates are renewed with a margin of at least a day. Shorter periods of replicator downtime will not break the cluster.

### Configuration

Replicator is configured in [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, apps/replicator directory.

### Control

To restart service run `kubectl rollout restart -n replicator deploy` for Kubernetes cluster or use ArgoCD UI to restart.

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

None.
