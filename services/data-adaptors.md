# Data Adaptors

## Summary

Data adaptors are used to order data from Airbus and Planet. Adaptors run in the EODH as user service workflows in specialised data provider namespaces via the ADES service.

There are separate adaptors for Planet, Airbus SAR, and Airbus Optical data providers. Each adaptor interfaces with the provider to place an order for a single item, wait for the item to be delivered to an S3 bucket, then downloads and adds assets to a STAC item tracking the order. The ADES process handles upload of assets and ingestion of STAC into the user's workspace.


### Code Repositories and Artifacts

- Code is available in https://github.com/EO-DataHub/commercial-data-adaptors repository

#### Airbus
- Container image published to public.ecr.aws/eodh/airbus-optical-adaptor and public.ecr.aws/eodh/airbus-sar-adaptor AWS ECR

#### Planet
- Container image published to public.ecr.aws/eodh/planet-adaptor AWS ECR


### Dependent Services

There are no services dependent on the adaptors.


## Operation

Adaptors run as an ADES workflow under either the `ws-planet` or `ws-airbus` namespace. Once deployed, they may be started by calls to an endpoint in the manage-catalogue-fastapi API.


### Configuration

Adaptors are configured as user service workflows via http and cwl scripts provided in the adaptor repository.


### Control

Adaptors are run and managed by the ADES service and the manage-catalogue-fastapi API. 

### Dependencies

- ADES - Adaptors run on the ADES service.
- Resource Catalogue stac-fastapi - Outputs of the adaptors are ingested into the ordering workspace's catalogue via stac-fastapi.
- Resource Catalogue manage-catalogue-fastapi - Adaptors are called by an API that handles inputs and creation of the order-tracking STAC item.
- Workspace Controller - Adaptors are dependent on data provider API keys linked to a workspace, and cannot place orders if a key is not linked.


### Backups

Outputs of the data adaptors are stored in S3 buckets. Ingested items are backed up as part of the stac-fastapi database. The original delivered items are not removed by the adaptors.


## Development

Adaptor code is version controlled in https://github.com/EO-DataHub/commercial-data-adaptors repository.

New versions are released by creating a release using the process described in the README of the repository.
