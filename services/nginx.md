# nginx

## Summary

nginx proxies all external traffic into the cluster, receiving requests via AWS's ELB (which in turn receives them from CloudFront).

nginx is unmodified third-party software and is installed using F5's nginx-ingress Helm chart (not ingress-nginx).

### Dependent Services

The hub will be entirely inaccessible without nginx.

### Configuration

nginx is configured in [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, apps/nginx directory.

### Control

To restart service run `kubectl rollout restart -n nginx deploy` for Kubernetes cluster or use ArgoCD UI to restart.

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

For all but always-public URLs auth-agent is required.
