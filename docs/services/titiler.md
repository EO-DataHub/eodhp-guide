# TiTiler

## Summary

TiTiler is a dynamic tile server used by the EO DataHub to provide on-the-fly map tile rendering for geospatial data. It generates map tiles from Cloud Optimized GeoTIFFs (COGs), Kerchunk, ZARR, NetCDF and other raster formats referenced in STAC (SpatioTemporal Asset Catalog) items within the Resource Catalogue. This allows users to visually explore data sets in a web map interface without pre-processing the data into a traditional tile cache.

### Code Repositories and Artifacts

- Microservice defined in a copy of the original TiTiler repository, located at https://github.com/EO-DataHub/titiler (A decision to copy the code rather than fork was made to allow for the custom auth implementation that wouldn't be appropriate for the original TiTiler repository).
- Microservice container image published to public.ecr.aws/eodh/titiler AWS ECR.
- Deployment is configured in the [ArgoCD Deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, in the apps/titiler directory.

### Dependent Services

- Any service that provides map-based visualisation of catalogue data, such as a Catalogue UI, will have degraded functionality if TiTiler is unavailable. Users will not be able to see visual previews of geospatial data.

## Operation

The service runs as a Kubernetes deployment named titiler under the titiler namespace. Traffic is routed to the service via the Nginx Ingress Controller, which makes the tiling endpoints available to front-end applications. The service receives requests that specify a STAC item or a COG URL and it generates and returns map tile images for display.

### Configuration

The TiTiler service is configured as part of the [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) in the apps/titiler directory.

### Control

To restart the service, run kubectl rollout restart -n titiler deployment titiler for the Kubernetes cluster or use the ArgoCD UI to restart.

To stop the service, it must be removed from the ArgoCD configuration.

### Dependencies

- **Nginx Ingress Controller:** Required to expose the TiTiler API endpoints externally.
- **S3 Object Storage:** Requires access to the S3 buckets where the source geospatial data (COGs/NetCDF/ZARR/Kerchunk) is stored.

### Backups

TiTiler is a stateless service. It does not manage or store any persistent data; it reads from data sources and generates tiles on-the-fly. Therefore, no backups are required for this service.

## Development

The TiTiler service is based on the open-source project [developmentseed/titiler](https://github.com/developmentseed/titiler). The EO DataHub version is maintained in the [EO-DataHub/titiler](https://github.com/EO-DataHub/titiler) repository.

New versions should be released by creating a new release using the GitHub web UI with a version tag following the pattern v1.2.3.