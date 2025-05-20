# Workflow Runner (ADES)

## Summary

The workflow runner is build on the EOEPCA ADES building block. It orchestrates workflow execution for the platform. A REST API has been built on top of the ADES component for managing workflow requests and integrating it with the platform IAM.

### Dependent Services

- Workflow executions will not work if the workflow runner is not operational
- Harvesting pipelines rely on the Workflow Runner to run

## Operation

The Workflow Runner is comprised of several services, which all run in the `ades` namespace of the Kubernetes cluster.

- deployment.apps/zoo-project-dru-zookernel
- deployment.apps/zoo-project-dru-zoofpm
- deployment.apps/zoo-project-dru-kubeproxy
- deployment.apps/workflow-runner-api
- deployment.apps/workflow-ingester
- TODO - any others relevant?

The `workflow-runner-api` exposes a REST API that receives requests and processes them.

TODO - how else can workflows be triggered? e.g. event sources?

### Configuration

The Auth Agent is configured as part of the [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) in the apps/ades directory.

### Control

To restart a Workflow Runner service run `kubectl rollout restart -n ades <service-fqdn>` for Kubernetes cluster or use ArgoCD UI to restart, where `service-fqdn` is one of the following, relating to the service to be restarted:

- deployment.apps/zoo-project-dru-zookernel
- deployment.apps/zoo-project-dru-zoofpm
- deployment.apps/zoo-project-dru-kubeproxy
- deployment.apps/workflow-runner-api
- deployment.apps/workflow-ingester
- TODO - any others relevant?

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

- Open Policy Agent - The Workflow Runner uses fine-grained authorisation policies defined in the Open Policy Agent. If the OPA is not available, authorisation of requests will fail.
- TODO - any more?

### Backups

All state for the Workflow Runner is in its database. Restoring a previous state involves following the database restore procedure.

## Development

### Code Repositories and Artifacts

The Workflow Runner is built from multiple microservices, each with their own code repository.

TODO - list all relevant code repositories (API, ADES, etc - might be more readable to split into sub-headings for the various services). Include any build artifact URIs, e.g. AWS ECR images.
