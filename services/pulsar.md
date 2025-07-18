# Pulsar

## Summary

Pulsar allows messages to be passed between services within the cluster. This provides functionality that is used throughout the harvest pipeline, workspaces API and accounting services among others.

### Code Repositories and Artifacts

- Pulsar deployment configuration controlled in [ArgoCD Deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, _apps/pulsar_ directory.

### Dependent Services

- Workspaces services - unable to create workspace data stores in S3 or Resource Catalogue
- Harvest pipeline - unable to transform or ingest data into the Resource Catalogue, including adapters
- Accounting - unable to receive and ingest new billing events from billing collectors, and won't receive information about workspace accounts from the workspace services.
- Other services that monitor topics or send messages using Pulsar

## Operation

Pulsar runs as a number of subservices in Kubernetes within the `pulsar` namespace. Deployed via a helm chart in the [ArgoCD Deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, _apps/pulsar_ directory.

Pulsar is a 3rd party open source software providing a distributed Publication/Subscription based messaging system as part of the EO DataHub platform.

Traffic external to the cluster is handled via an ingress and allows access to the Pulsar UI, available at https://pulsar.eodatahub.org.uk to authenticated users.

You can make use of the Pulsar API to monitor Pulsar health across topics, by port-forwarding into one of the Pulsar pods and following the Pulsar API [guide](https://pulsar.apache.org/docs/4.0.x/pulsar-api-overview/).


### Configuration

Pulsar is configured in [ArgoCD Deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, apps/pulsar directory. Deployed via Helm chart with a values file for configuration.

### Control

To restart the stac-fastapi service run `kubectl rollout restart -n pulsar <service-fqdn>` for Kubernetes cluster or use ArgoCD UI to restart, where `service-fqdn` is one of the following, relating to the service to be restarted:

- deployment.apps/pulsar-bookie
- deployment.apps/pulsar-broker
- deployment.apps/pulsar-proxy
- deployment.apps/pulsar-pulsar-manager
- deployment.apps/pulsar-recovery
- deployment.apps/pulsar-zookeeper

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

- Nginx Ingress Controller must be working to access Pulsar through the web UI

### Backups

Pulsar stores its data in persistent volumes as part of its deployment. Pulsar is configured with statefulsets and replicas to handle occasions where services become unresponsive to prevent data loss. No other backups are required.

## Development

Pulsar is a 3rd-party open-source project, https://github.com/apache/pulsar. Stock images are available at apachepulsar/pulsar-all. This is deployed via Helm chart, https://pulsar.apache.org/charts, with configuration handled via a values file with Kustomize.

Updates can be made to the configuration in the ArgoCD deployment repository, https://github.com/EO-DataHub/eodhp-argocd-deployment, in the apps/pulsar directory, and then raised as a pull requests against the main branch.