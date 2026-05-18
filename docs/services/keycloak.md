# Keycloak

## Summary

Keycloak is the OIDC IdP for the platform. It is the authentication service and manages platform OIDC clients and allows for single sign-on among managed clients.

### Code Repositories and Artifacts

- Uses open source Keycloak images as a base (https://github.com/keycloak/keycloak)
- Custom plugins maintained in https://github.com/EO-DataHub/eodh-keycloak
- Custom build images stored in AWS ECR public.ecr.aws/eodh/eodh-keycloak
- Deployment configured in https://github.com/EO-DataHub/eodhp-argocd-deployment repository, apps/keycloak directory
- Keycloak theme modified using https://github.com/EO-DataHub/keycloakify-starter.git and theme.jsr artifact installed in https://github.com/EO-DataHub/eodh-keycloak

### Dependent Services

- Oauth2 Proxy will fail authenticate requests
- Sign-in to hub will fail
- Keycloak admin panel will be unavailable
- Any service that calls Keycloak API will fail (e.g. Auth Agent, Workflow Runner, Workspace Services)

## Operation

Keycloak is the authentication backbone of the platform. It maintains the database of users and their associated RBAC.

Requests are made to the Keycloak API via the platform OIDC clients. If Keycloak is unavailable then authentication requests will all fail, causing major disruption to the platform.

Keycloak runs as a stateful set, `keycloak`, in Kubernetes in the `keycloak` namespace. 

### Configuration

Keycloak is configured in [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, apps/keycloak directory.

### Control

To restart service run `kubectl rollout restart -n keycloak statefulset keycloak ` for Kubernetes cluster or use ArgoCD UI to restart.

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

None

### Backups

All state for the Keycloak is in its database. Restoring a previous state involves following the database restore procedure.

## Development

Keycloak is an 3rd-party open-source project, https://github.com/keycloak/keycloak. Stock images are available at quay.io/keycloak/keycloak.

This project has developed custom plugins for Keycloak, which are managed at https://github.com/EO-DataHub/eodh-keycloak. This plugins are integrated into the open-source keycloak image and published to public.ecr.aws/eodh/eodh-keycloak.

New plugin versions, or updates to the Keycloak base image, should be released by creating a new release in https://github.com/EO-DataHub/eodh-keycloak using GitHub web UI with a version tag following the pattern v<keycloak-base-version>-<plugin-version> The commit tag will trigger the GitHub action release process.

Alternately, releases may be published directly from the code repository with `make publish version=v26.0.4-1.2.3`, but this should only be used for test releases as the Git commit will not be properly tagged.
