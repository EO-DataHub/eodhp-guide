# EO DataHub OpenAPI Docs

## Summary

This service collects together all of the platform microservice self-published OpenAPI docs and presents them to the user a single documentation page for the platform and workspace domains.

The OpenAPI sources can be defined as either http[s] links or local files.

The HTML view of the docs are available at:

- eodatahub.org.uk/api/docs
- {workspace}.eodatahub-workspaces.org.uk

### Code Repositories and Artifacts

- Microservice defined in https://github.com/EO-DataHub/eodh-openapi repository
- Microservice container image published to public.ecr.aws/eodh/eodh-openapi AWS ECR
- Deployment is configured in https://github.com/EO-DataHub/eodhp-argocd-deployment repository, apps/docs directory

### Dependent Services

- None

## Operation

The openapi-docs service runs as two separate Kubernetes deployments named `platform-docs` and `workspace-docs` under the `docs` namespace. `platform-docs` manages the OpenAPI docs for the eodatahub.org.uk domain and `workspace-docs` for the eodatahub-workspaces.org.uk domain.

### Configuration

The OpenAPI Docs are configured as part of the [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) in the apps/docs directory.

### Control

To restart service run `kubectl rollout restart -n docs deployment platform-docs`, `kubectl rollout restart -n docs deployment workspace-docs` for Kubernetes cluster or use ArgoCD UI to restart.

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

- Any microservice that publishes its own OpenAPI docs. If the microservice is not available when the openapi-docs services starts then it will not be included in the documentation. It will not fail due to missing OpenAPI sources.

### Backups

The OpenAPI docs hold no state, it only references configuration from Kubernetes ConfigMaps (version controlled as part of ArgoCD repo) or OpenAPI specs hosted by microservices.

## Development

The OpenAPI Docs service code is version controlled in https://github.com/EO-DataHub/eodh-openapi repository.

New versions should be released by creating a new release using GitHub web UI with a version tag following the pattern v1.2.3. The commit tag will trigger the GitHub action release process.

Alternately, releases may be published directly from the code repository with `make publish version=v1.2.3`, but this should only be used for test releases as the Git commit will not be properly tagged.
