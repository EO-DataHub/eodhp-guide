# ArgoCD Web UI

## Purpose

The ArgoCD web UI can be useful to get an overview of platform health. It is recommended to consult the web UI if there are any issues with the platform.

## When to Use

The ArgoCD UI should be one of the first checks to determine platform health. Use it to check that all services are working as expected.

## Operation

Access to the web UI can be either through:

1. Visiting https://argocd.eodatahub.org.uk
2. Port forwarding the web UI to a local port

Visiting https://argocd.eodatahub.org.uk is the preferred option, and is generally more responsive and stable. Using local port forwarding is useful for when the ArgoCD UI is down, e.g. during platform malfunction.

### Access https://argocd.eodatahub.org.uk

You will need credentials to access the ArgoCD UI. You can sign in using Keycloak, but you must have user credentials for the master realm and that user must have the "admin" realm role. Alternately you can get the admin credentials from the Kubernetes cluster.

### Local Port-Forward

If the ArgoCD web UI is not available at https://argocd.eodatahub.org.uk then, provided you have access to the Kubernetes cluster, you can port forward the web UI to a local port.

```sh
kubectl port-forward service/argocd-server -n argocd 8080:443
```

You can now visit http://127.0.0.1:8080 to view the web UI. You will not be able to log in using Keycloak for this method, and must use the ArgoCD admin user credentials.

### Obtaining ArgoCD Admin Credentials from Kubernetes Cluster

You can obtain the ArgoCD admin password from the Kubernetes cluster using `kubectl`. The username is `admin`.

```sh
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

## Requirements

- An account in Keycloak master realm with "admin" realm role OR access to the Kubernetes cluster via `kubectl`.

## Useful Information

- Just because a service is healthy in ArgoCD doesn't mean it is working as expected. A service can be showing as healthy but be failing to execute its duties.
