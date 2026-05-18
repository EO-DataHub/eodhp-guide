# Workspaces

## Summary

This service manages the workspaces available on the DataHub. This includes creation and deletion requests when a new user creates a Hub account or chooses to deleted their data from the Hub. It also reconciles associated resources, such as User roles and object and file storage in AWS.

### Code Repositories and Artifacts

The Workspaces service is built from multiple microservices, each with their own code repository.
- Workspace services: https://github.com/EO-DataHub/eodhp-workspace-services
  - Managed in ArgoCD repository in directory apps/workspaces/base/api
  - Image: public.ecr.aws/eodh/zoo-project-dru/eodhp-workspace-services
- Workspace controller: https://github.com/EO-DataHub/eodhp-workspace-controller
  - Managed in ArgoCD repository in directory apps/workspaces/base/controller
  - Image: public.ecr.aws/eodh/workspace-controller
  - Helm: public.ecr.aws/eodh/helm/workspace-operator
- Workspace efs access: deployed using nginx
  - Managed in ArgoCD repository in directory apps/workspaces/base/efs-files
- Workspace manager: https://github.com/EO-DataHub/eodhp-workspace-manager
  - Managed in ArgoCD repsitory in directory apps/workspaces/base/manager
  - Image: public.ecr.aws/eodh/eodhp-workspace-manager

### Dependent Services

Any services that require workspace access will not function correctly, unless the underlying workspace has been correctly reconciled by the workspaces service. 
- For example, any services that require authenticated access to AWS, such as via S3 Object store, may be rejected, as the required IAM role may not exist in AWS. The workspaces services also generate workpace sub-catalogs in the Resource Catalogue, so this could impact the Resource Catalogue and STAC-FastApi services as well. Other impacted services could include the Workflow Runner, S3 and EFS data access, and Jupyter Notebooks.
- Keycloak also depends on the workspace services API to determine which workspaces and accounts a user has access to, so will be unable to issue new tokens for a user without the service. This means the Hub will only display public web pages will any that require authentication failing to load.
- The accounting service depends on the workspace manager sending workspace status messages via Pulsar in order to record associations between workspaces and accounts.
- The workspace UI depends on the workspace services API to provide information about workspaces and accounts, so this page will fail to populate should the API be unavailable.

## Operation

The Workspaces service runs as a set of deployments, in Kubernetes in the `workspaces` namespace. This generates user namespaces of the form `ws-<workspace-name>`.

Traffic reaches the workspace API via an ingress, available at `/api/workspaces`, this endpoint allows users to list their available workspaces, list, add and remove users from a workspace and generate workspace-scoped credentials. The Workspace services also provide an API at `/api/accounts` which can be used to view available billing account information.

Traffic is also routed to the workspace efs service via another ingress, which allows users to access data stored in EFS inside their workspace.

Workspaces can be deleted by workspace owners using the Workspace UI.

Logs for the workspace services can be viewed using the ELK Stack UI at https://logs.eodatahub.org.uk, searching for `kubernetes.namespace : "workspaces"`. You can also view logs in the workspace-controller-manager and workspace-manager pods either in the ArgoCD UI or using the Kubernetes CLI `kubectl -n workspaces logs <pod-name>`. Logs for the efs service can be viewed in the `efs-nginx` pod.

Workspace data is stored in the workspaces-db in the `databases` namespace. This can be viewed using Postgres applications such as pg-admin.

### Configuration

The Workspaces service is configured in [ArgoCD Deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, apps/workspaces directory. The workspace controller is deployed via Helm chart with the values file at apps/workspaces/base/controller/controller.

### Control

To restart a Workflow Runner service run `kubectl rollout restart -n workspaces <service-fqdn>` for Kubernetes cluster or use ArgoCD UI to restart, where `service-fqdn` is one of the following, relating to the service to be restarted:

- efs-nginx
- workspace-controller-manager
- workspace-manager
- workspace-services

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

This services depends on the following services:
- Pulsar service to support creation of workspace sub-catalogs in the Resource Catalogue and also for communication between Workspace Services and Workspace Manager
- Databases service to store workspace details
- AWS access to configure IAM roles, S3 and EFS storage

### Backups

All state for the Workspaces service is in its database. Restoring a previous state involves following the database restore procedure. The database is deployed as part of the databases service.

## Development

Code is stored in Github in the following repositories:
- Workspace services: https://github.com/EO-DataHub/eodhp-workspace-services
- Workspace controller: https://github.com/EO-DataHub/eodhp-workspace-controller
  - Deployed via Helm chart, also generated from this repository
- Workspace manager: https://github.com/EO-DataHub/eodhp-workspace-manager

The workspace efs-service uses the official public nginx image which is maintained by a 3rd party as open source software.

The resource code is version controlled in the repositories stated above.

New versions should be released by creating a new release using GitHub web UI with a version tag following the pattern v1.2.3. The commit tag will trigger the GitHub action release process.
