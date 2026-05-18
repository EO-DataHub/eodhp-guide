# Open Policy Agent

## Summary

Open Policy Agent controls the course and fine grained authorisation for the EO DataHub.

### Code Repositories and Artifacts

- Open Policy Agent Helm chart in https://github.com/permitio/opal-helm-chart
- Policy version controlled in https://github.com/EO-DataHub/eodhp-opa-config. Policies for different environments controlled on different branches, e.g. `eodhp-staging` and `eodhp-prod`. Branch is referenced in ArgoCD configuration repo.
- Deployment configured in https://github.com/EO-DataHub/eodhp-argocd-deployment repository, _apps/opal_ directory.

### Dependent Services

- Auth Agent will fail to authorise any requests to the platform that require an Nginx auth_request
- ADES will fail to perform fine grained authorisation

## Operation

Open Policy Agent runs as a client/server pair of deployments, `opal-server` and `opal-client`, in Kubernetes in the `opal` namespace. The server is responsible for syncing its internal policy with the Policy repo and distributing to clients. Clients provide an API to query policy.

OPA clients are called by Auth Agent (course grained authorisation) or upstream clients (e.g. ADES) for fine grained authorisation.

### Configuration

OPA deployment is configured in [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, _apps/opal_ directory.

Actual OPA policy is controlled on specific branches of [Policy](https://github.com/EO-DataHub/eodhp-opa-config) repo. Find the relevant branch by inspecting _apps/opal/envs/$ENV/kustomization.yaml_ patches in ArgoCD deployment repo to see branch environment points to.

### Control

To restart service run `kubectl rollout restart -n opal deploy opal-server` for Kubernetes cluster or use ArgoCD UI to restart.

Similarly for `kubectl rollout restart -n opal deploy opal-client`.

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

- Policy repo must be available to the `opal-server`, either publically or credentials must be configured through AWS secret store.

### Backups

Although it looks like `opal-server` uses a PostgreSQL database that the Helm chart deploys with it, this is only used for messaging to distribute the policy from the sevrer to the clients. No state will be lost if this data is lost as it will be rebuilt from the policy repo.

## Development

Open Policy Agent is 3rd party open source software and is unmodified for this project.

Modify the Open Policy Agent policy in policy repo and push changes to branch relevant to the environment you want to update.
