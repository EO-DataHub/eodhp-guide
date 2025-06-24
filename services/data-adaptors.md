# Data Adaptors

## Summary

Data adaptors enable ordering of commercial data from Airbus and Planet within the EO DataHub. These adaptors run as user service workflows in specialised data provider namespaces via the ADES service.

Each adaptor interfaces with its respective provider to place an order for a single item, waits for delivery to an S3 bucket, then downloads and attaches assets to a STAC item that tracks the order. The ADES process manages asset upload and ingestion of STAC items into the user's workspace.

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

Adaptors run as ADES workflows under either the `ws-planet` or `ws-airbus` namespaces. Once deployed, they can be started by making calls to an endpoint in the `manage-catalogue-fastapi` API.

### Configuration

Adaptors are configured as user service workflows using HTTP and CWL scripts provided in the adaptor repository.

### Control

Adaptors are managed and executed by the ADES service and the `manage-catalogue-fastapi` API.

### Dependencies

- **ADES:** Adaptors run on the ADES service.
- **Resource Catalogue (`stac-fastapi`):** Outputs from the adaptors are ingested into the ordering workspace's catalogue.
- **Resource Catalogue (`manage-catalogue-fastapi`):** Adaptors are triggered by this API, which handles inputs and the creation of the order-tracking STAC item.
- **Workspace Controller:** Adaptors require data provider API keys linked to a workspace. Orders cannot be placed if a key is not linked.

### Backups

Adaptor outputs are stored in S3 buckets. Ingested items are backed up as part of the `stac-fastapi` database. The original delivered items are not removed by the adaptors.

## Development

Adaptor code is version controlled in the [EO-DataHub/commercial-data-adaptors](https://github.com/EO-DataHub/commercial-data-adaptors) repository.

New versions are released by following the release process described in the repository's README.
