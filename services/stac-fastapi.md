# STAC-FastApi

## Summary

This service handles the STAC Catalogue for EO DataHub, using the STAC-FastApi package. This service provides an API for searching STAC Catalogs, Collections and Items using GET and POST requests. This service also includes an ingester which parses input STAC data and access policies from S3 and ingests them into the STAC-FastApi deployment. Another job is included which harvests and ingests the initial top-level catalogs into which all sub-catalogs will be ingested.

### Code Repositories and Artifacts

- The STAC-FastApi deployment for DataHub uses an Elasticsearch backend and is built from two repositories:
  - The parent package is configured in the [eodhp-stac-fastapi](https://github.com/EO-DataHub/eodhp-stac-fastapi) repository 
  - The elasticsearch implementation is configured in the [eodhp-stac-fastapi-elasticsearch-opensearch](https://github.com/EO-DataHub/eodhp-stac-fastapi-elasticsearch-opensearch repository)
- The STAC-FastApi ingester is configured in the [stac-fastapi-ingester](https://github.com/EO-DataHub/stac-fastapi-ingester) repository
- The Initial Catalog Harvester is configured in the [eodhp-init-catalogs-harvest](https://github.com/EO-DataHub/eodhp-init-catalogs-harvest) repository
- The STAC-FastApi service is deployed as a public image available at public.ecr.aws/eodh/eodhp-stac-fastapi
- The STAC-FastApi Ingester service is deployed as a public image available at public.ecr.aws/eodh/eodhp-stac-fastapi-ingester
- The Initial catalog harvest job is deployed as a public image available at public.ecr.aws/eodh/eodhp-init-catalog-ingest


### Dependent Services

- The Resource Catalogue UI uses the STAC-FastApi backend to retrieve STAC data from the Hub
- Workflows that use inputs from the STAC Catalogue will fail as they cannot read these inputs from the API
- Any applications or notebooks that rely on the STAC Catalogue will fail as they cannot access STAC data from the Hub
- The Harvest Pipeline will fail, as the STAC data cannot be ingested correctly and so any harvested data will not be available on the Hub

## Operation

This service runs in Kubernetes as three services, one being read-only for external client access, run under `catalogue-search-service-client` in namespace `sfapi2`, and two providing read-write access for the ingester, run under `catalogue-search-service-ingester` and `catalogue-search-service-ingester-bulk` in namespace `sfapi2`, the first handling workspace uploads and workflow outputs, and the latter handling larger ingester requests from the harvest pipeline.

All services rely on an elasticsearch deployment.

Traffic is routed to the client via an ingress, available at `/api/catalogue/stac`

The ingester services rely on Pulsar messaging to ingest new STAC data from S3 file keys. The ingester and ingester-bulk listen to the `transformed` and `transformed-bulk` topics respectively.

A job also runs on application startup which generates and ingests the top-level catalogs into the catalogue for `public`, `commercial` and `user` which are required before any other catalogs can be ingested.

### Configuration

The service is configured in the ArgoCD deployment repo, apps/stac-fastapi-2 directory. The elasticsearch backend is deployed via Helm Chart and can be configured using the values.yaml file in the apps/stac-fastapi-2/base directory.

### Control

To restart the stac-fastapi service run `kubectl rollout restart -n sfapi2 <service-fqdn>` for Kubernetes cluster or use ArgoCD UI to restart, where `service-fqdn` is one of the following, relating to the service to be restarted:

- deployment.apps/catalogue-search-service-client
- deployment.apps/catalogue-search-service-ingester
- deployment.apps/catalogue-search-service-ingester-bulk
- deployment.apps/elasticsearch-master

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

- The elasticsearch instance must be running in this namespace for any of the stac-fastapi services to be operational
- Relies on auth-agent to provide auth header tokens when attempting to view private datasets

### Backups

Where is state of service backed up? How to restore?

## Development

The stac-fastapi deployments are developed across two repositories:
- The first defines the abstract methods for STAC-FastApi, found in the [eodhp-stac-fastapi](https://github.com/EO-DataHub/eodhp-stac-fastapi) repository
- The second defines the elasticsearch implementation for STAC-FastApi, found in the [eodhp-stac-fastapi-elasticsearch-opensearch](https://github.com/EO-DataHub/eodhp-stac-fastapi-elasticsearch-opensearch) repository

The ingester is configured in the [stac-fastapi-ingester](https://github.com/EO-DataHub/stac-fastapi-ingester) repository.

The job to ingest the initial top-level catalogs is configured in the [eodhp-init-catalogs-harvest](https://github.com/EO-DataHub/eodhp-init-catalogs-harvest) repository.

Where is the code for this service kept?

What is the release process?
