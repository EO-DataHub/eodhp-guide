# Oauth2 Proxy

## Summary

OAuth2 Proxy instances act as OIDC clients to Keycloak for user access to the EO DataHub. There is one Oauth2 Proxy deployment per domain, platform (eodatahub.org.uk) and workspaces (eodatahub-workspaces.org.uk). The workspaces deployment handles both eodatahub-workspaces.org.uk and \*.eodatahub-workspaces.org.uk domains.

Both OAuth2 Proxy deployments are Keycloak confiential clients (they require a client ID and client secret to authenticate with Keycloak, the secrets are defined by the AWS secret store).

### Code Repositories and Artifacts

- Uses [OAuth2 Proxy Helm chart](https://artifacthub.io/packages/helm/oauth2-proxy/oauth2-proxy)

### Dependent Services

- The web presence will not allow users to log in to the relevant domain if any OAuth Proxy deployment is down

## Operation

Both OAuth2 Proxy deployments run in the Kubernetes namespace `oauth2-proxy`.

Traffic reaches the deployments through Nginx Ingress Controller. Login requests are sent to respective OAuth2 Proxy instances, which starts the OIDC login process with Keycloak.

### Configuration

OAuth2 Proxies are configured in [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, apps/oauth2-proxy directory.

### Control

To restart platform OAuth2 Proxy, run `kubectl rollout restart -n oauth2-proxy deployment oauth2-proxy-platform` for Kubernetes cluster or use ArgoCD UI to restart.

Similarly for workspace OAuth2 Proxy, `kubectl rollout restart -n oauth2-proxy deployment oauth2-proxy-workspaces`.

To stop service, the app must be removed from ArgoCD configuration.

### Dependencies

- Keycloak - Needs to read /.well-known/openid-configuration from Keycloak on start up, therefore will not start until Keycloak is up
- Redis - required to store users' session info

### Backups

No state managed that requires backup. Redis holds only state, and it is not disasterous if this is lost. User (cookie) session state will be lost, requiring users to re-login.

## Development

3rd party open source software. Keep an eye out for new releases and update the ArgoCD repo with new version numbers as required.
