# Linkerd

## Summary

Linkerd provides a service mesh. Pods and Services in the mesh can communicate with mTLS and authorization policies can be created in Kubernetes which determine which pods can communicate with which other pods. Linkerd starts a sidecar container in each meshed pod which uses iptables to intercept all incoming and outgoing network traffic.

EODH uses linkerd to prevent user workloads from connecting to internal services not designed for receiving untrusted traffic. This currently works by keeping user workloads outside the mesh and configuring internal workloads to only accept connections from other meshed pods.

### Code Repositories and Artifacts

Source and Helm chart for linkerd is at https://github.com/linkerd/linkerd2/

### Dependent Services

All user-facing services will fail if linkerd fails completely. See [Bootstrap Dependencies](../operations/bootstrap-dependencies.md) - anything listed before linkerd (plus ArgoCD) will not depend on it. These non-dependencies are only core Kubernetes infrastructure such as cert-manager or the autoscaler.

## Operation

linkerd has several parts:

- The linkerd proxy injector (running in the linkerd namespace) provides a webhook that injects the linkerd sidecar into new pods. If this fails then creation or modification of Pods or Deployments in any namespace not listed as excluded in `eodhp-argocd-deployment/apps/linkerd/base/control-plane/values.yaml` will fail - currently those listed are kube-system, cert-manager, replicator, secret-gen, argocd, autoscaler, aws and certs.
- The linkerd identity service (running in the linkerd namespace) issues certificates for mTLS. Meshed pods will not start if this is not working.
- linkerd-destination in the linkerd namespace distributes linkerd policy data. Again, meshed pods may not start if this is not working.
- Various other pods in the linkerd namespace provide the web UI and monitoring and are not depending on by other services.
- linkerd sidecars running inside workloads pods can fail, bringing down only the service they're running in. This is most likely to happen if the linkerd root certificate expires, in which case it must be renewed manually and all meshed pods must be restarted. See [Rotating Linkerd Trust Anchor](../operations/maintenance/rotating-linkerd-trust-anchor.md) for the procedure.

The linkerd web UI can be accessed using `kubectl port-forward svc/web -n linkerd 8084:8084` and connecting to localhost:8084. This provides information about meshed traffic in the cluster. Traffic between meshed pods can be intercepted and viewed here, for example to inspect the HTTP requests being made between internal services.

### Configuration

Linkerd is configured in [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, in apps/linkerd for the global configuration and as Linkerd Kubernetes resources (Server, HTTPRoute, NetworkAuthentication, AuthorizationPolicy), typically in a linkerd-policy.yaml file, for service-specific authorization configuration.

### Control

To restart service run `kubectl rollout restart -n linkerd deploy` for Kubernetes cluster or use ArgoCD UI to restart. Sidecars can only be restarted by restarting the Pod they're part of.

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

None

### Backups

Not applicable.

## Development

Linkerd is a third-party open source project which we have not modified.
