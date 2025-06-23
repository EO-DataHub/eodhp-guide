# Docs

## Summary

A compilation of all OpenAPI documentation for the services that provide an API, including STAC-FastApi, Accounting and the Workflow Runner.

### Code Repositories and Artifacts

- EODHP OpenAPI deployment, controlled in [eodh-openapi](https://github.com/EO-DataHub/eodh-openapi) repository
- EODHP OpenAPI configuration, controlled in [ArgoCD Deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, apps/docs.
- OpenAPI docker image available at [public.ecr.aws/eodh/eodh-openapi](public.ecr.aws/eodh/eodh-openapi)

### Dependent Services

- Combined OpenAPI documentation will not be available from the web browser

## Operation

The service runs as two deployments, `platform-docs` and `workspace-docs`, in Kubernetes within the `docs` namespace, providing webpages combining all the hub APIs into a single OpenAPI documentation page - one for platform endpoints and another for workspace-specific endpoints. Services, `platform-docs` and `workspace-docs`, work with ingresses of the same name to allow external traffic so that users can request the OpenAPI documentation webpage and make API calls using the UI. The API also uses browser cookies to provide authenticated access to endpoints that require Authorization headers for certain functionality.

### Configuration

The docs API is configured in [ArgoCD Deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, apps/docs directory.

### Control

To restart service run `kubectl rollout restart -n docs deployment platform-docs` or `kubectl rollout restart -n docs deployment workspace-docs` for Kubernetes cluster or use ArgoCD UI to restart.

When any of the Hub APIs has been updated, altering any endpoint definitions, the combined OpenAPI docs will also need to be restarted to gather and deploy these changes to the `/api/docs` endpoint. To handle these changes, the `docs` service must be restarted using the above commands, depending on what sub-service has been altered.

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

This service works by combining all the APIs provided by the Hub into a single OpenAPI doc, therefore, if any of these services is unavailable, the resulting OpenAPI docs may exclude some endpoints. The dependent services are:
- Workflow Runner API
- Accounting API
- Resource Catalogue API
- TiTiler API
- Billing and Accounts API
- Workspace Management API

### Backups

No backup required.

## Development

The code for this service is managed in [eodh-openapi](https://github.com/EO-DataHub/eodh-openapi). A docker image is built from this reposuitory and published to AWS ECR public.ecr.aws/eodh/eodh-openapi.

The image is then configured in [ArgoCD Deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, apps/docs directory, in the deployments under both the paltform and workspaces directories.
