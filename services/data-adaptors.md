# Data Adaptors

## Summary

Data adaptors enable ordering of commercial data from Airbus and Planet within the EO DataHub. These adaptors run as user service workflows in specialised data provider workspaces via the [Workflow Runner](workflow-runner.md) service.

Each adaptor interfaces with its respective provider to place an order for a single item, waits for delivery to an S3 bucket, then downloads and attaches assets to a STAC item that tracks the order. The Workflow Runner manages asset upload and ingestion of STAC items into the user's workspace.

### Code Repositories and Artifacts

- Source code: [EO-DataHub/commercial-data-adaptors](https://github.com/EO-DataHub/commercial-data-adaptors)

**Airbus:**
- Container images:
  - `public.ecr.aws/eodh/airbus-optical-adaptor`
  - `public.ecr.aws/eodh/airbus-sar-adaptor`

**Planet:**
- Container image:
  - `public.ecr.aws/eodh/planet-adaptor`

### Dependent Services

There are no services that depend on the adaptors.

## Operation

Adaptors run as workflows under either the `ws-planet` or `ws-airbus` namespaces. Once deployed, they can be started by making calls to an endpoint in the `manage-catalogue-fastapi` API.

### Configuration

Adaptors are configured as user service workflows using HTTP and CWL scripts provided in the adaptor repository.

### Control

Adaptors are managed and executed by the Workflow Runner service and the `manage-catalogue-fastapi` API in the Resource Catalogue.

### Dependencies

- **Workflow Runner:** Adaptors run as workflows within the EODH.
- **STAC FastAPI:** Outputs from the adaptors are ingested into the ordering workspace's catalogue.
- **Resource Catalogue (`manage-catalogue-fastapi`):** Adaptors are triggered by this API, which handles inputs and the creation of the order-tracking STAC item.
- **Workspace Controller:** Adaptors require data provider API keys linked to a workspace. Orders cannot be placed if a key is not linked.

### Backups

Adaptor outputs are stored in S3 buckets. Ingested items are backed up as part of the `stac-fastapi` database. The original delivered items are not removed by the adaptors.

## Development

Adaptor code is version controlled in the [EO-DataHub/commercial-data-adaptors](https://github.com/EO-DataHub/commercial-data-adaptors) repository.

New versions are released by following the release process described in the repository's README. Deploying adaptors is a manual one time process that must be done when all dependencies are deployed and workspaces for data providers are created.

In order to deploy adaptors, `planet` and `airbus` workspaces must first exist. It is useful but not necessary to make a specific user to own each of these workspaces to maintain security and allow admins to deploy the adaptors with a workspace scoped token. Login credentials for these accounts can be managed via keycloak.
