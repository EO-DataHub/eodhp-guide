# EO DataHub OpenAPI Docs

## Summary

This service collects together all of the platform microservice self-published OpenAPI docs and presents them to the user a single documentation page for the platform and workspace domains.

The OpenAPI sources can be defined as either http[s] links or local files.

The HTML view of the docs are available at:

- eodatahub.org.uk/api/docs
- {workspace}.eodatahub-workspaces.org.uk

### Code Repositories and Artifacts

- Microservice defined in [eodh-openapi](https://github.com/EO-DataHub/eodh-openapi)
- Microservice container image published to [public.ecr.aws/eodh/eodh-openapi](public.ecr.aws/eodh/eodh-openapi) AWS ECR
- Deployment is configured in [ArgoCD Deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, apps/docs directory

### Dependent Services

- No dependent services, but combined documentation will not be available from the web browser

## Operation

The openapi-docs service runs as two separate Kubernetes deployments: `platform-docs` and `workspace-docs`, both under the `docs` namespace.  
- `platform-docs` serves the OpenAPI documentation for the main eodatahub.org.uk domain.
- `workspace-docs` serves documentation for the workspace domain (eodatahub-workspaces.org.uk).

Each deployment provides a single documentation page for its respective domain, aggregating the OpenAPI specs from all relevant microservices.  
Both services, `platform-docs` and `workspace-docs`, work with ingresses of the same name to allow external traffic so that users can request the OpenAPI documentation webpage and make API calls using the UI. The API also uses browser cookies to provide authenticated access to endpoints that require Authorization headers for certain functionality.

### Configuration

The OpenAPI Docs are configured as part of the [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) in the apps/docs directory.

### Control

To restart service run `kubectl rollout restart -n docs deployment platform-docs` or `kubectl rollout restart -n docs deployment workspace-docs` for Kubernetes cluster or use ArgoCD UI to restart.

When any of the Hub APIs has been updated, altering any endpoint definitions, the combined OpenAPI docs will also need to be restarted to gather and deploy these changes to the `/api/docs` endpoint. To handle these changes, the `docs` service must be restarted using the above commands, depending on what sub-service has been altered.

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

This service works by combining all the APIs provided by the Hub into a single OpenAPI doc, therefore, if any of these services is unavailable, the resulting OpenAPI docs may exclude some endpoints. The dependent services include:
- Workflow Runner API
- Accounting API
- Resource Catalogue API
- TiTiler API
- Billing and Accounts API
- Workspace Management API

### Backups

The OpenAPI docs hold no state, it only references configuration from Kubernetes ConfigMaps (version controlled as part of ArgoCD repo) or OpenAPI specs hosted by microservices.

## Development

The OpenAPI Docs service code is version controlled in [eodh-openapi](https://github.com/EO-DataHub/eodh-openapi).

New versions should be released by creating a new release using GitHub web UI with a version tag following the pattern v1.2.3. The commit tag will trigger the GitHub action release process.

Alternately, releases may be published directly from the code repository with `make publish version=v1.2.3`, but this should only be used for test releases as the Git commit will not be properly tagged.

The image is then configured in [ArgoCD Deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, apps/docs directory, in the deployments under both the platform and workspaces directories.
