# Resource Catalogue

## Summary

The resource catalogue contains services relating to data access and harvesting from source, along with transformations applied to the data and ordering data from commercial sources.

The resource catalogue currently supports the following data sources:

- STAC catalogues
- Airbus
- Planet


### Code Repositories and Artifacts

- Deployment is configured in https://github.com/EO-DataHub/eodhp-argocd-deployment repository, apps/resource-catalogue directory

#### Data sources

##### SPDX harvester
- The SPDX harvester is a cron job that runs monthly to compare the already fetched files with those available at SPDX licenses. It first checks for a valid SPDX license identifier. If one is found, it creates two different license links using the transformer code repository, which then become part of the collection. Currently, the licenses are stored in an SPDX S3 bucket for each environment, and the bucket is hosted using CloudFront.
- Code available in https://github.com/EO-DataHub/eodhp-spdx-change-scanner repository
- Container image published to public.ecr.aws/eodh/eodhp-spdx-change-scanner AWS ECR

##### Airbus harvester
- Harvests data from Airbus and converts to STAC format
- Code available in https://github.com/EO-DataHub/airbus-harvester repository
- Container image published to public.ecr.aws/eodh/airbus-harvester AWS ECR

##### Planet harvester
- Harvests data for Planet collections (only) and converts to STAC format
- Code available in https://github.com/EO-DataHub/planet-harvester repository
- Container image published to public.ecr.aws/eodh/planet-harvester AWS ECR

##### STAC harvester
- Harvests STAC data from STAC catalogues
- The STAC harvester is configured in the https://github.com/EO-DataHub/stac-harvester-configurations repository. 
- Ingesting of STAC harvesters is carried out by ingesters. Code available in the https://github.com/EO-DataHub/stac-harvester-ingester repository.
- STAC harvester ingester container image published to public.ecr.aws/eodh/stac-harvester-ingester AWS ECR
- STAC harvester code available in the https://github.com/EO-DataHub/stac-harvester repository
- STAC harvester ingester container image published to public.ecr.aws/eodh/stac-harvester AWS ECR

##### Workspace file harvester
- Harvests user-supplied files into the user data directory
- Code available in https://github.com/EO-DataHub/workspace-file-harvester repository
- Container image published to public.ecr.aws/eodh/workspace-file-harvester AWS ECR

##### Planet proxy
- Proxy for Planet to access data and convert to STAC format
- Code available in https://github.com/EO-DataHub/stac-planet-api repository
- Container image published to public.ecr.aws/eodh/stac-planet-api AWS ECR


#### Transformers

##### Harvest
- Transforms STAC content into a standardised EODH format, ensuring correct links, summaries, and extensions where applicable.
- Code available in https://github.com/EO-DataHub/harvest-transformer repository
- Container image published to public.ecr.aws/eodh/harvest-transformer AWS ECR

##### Annotations
- Handles annotations data to be ingested into the the resource catalogue
- Code available in https://github.com/EO-DataHub/annotations-transformer repository
- Container image published to public.ecr.aws/eodh/annotations-transformer AWS ECR


#### FastAPI
- API to allow interaction with STAC-FastAPI within the EO DataHub.
- Code available in https://github.com/EO-DataHub/resource-catalogue-fastapi repository
- Container image published to public.ecr.aws/eodh/resource-catalogue-fastapi AWS ECR


### Dependent Services

The elasticsearch ingester takes inputs from the harvest transformer. 

To generate an API key for elasticsearch:

- Go to https://logs.eodatahub.org.uk
- Click `Elasticsearch`
- Click `Endpoints & API keys` (top right)
- Click on the `API key` tab
- Create an API key and add it to the `resource-catalogue.workspaces.elasticsearch.api_key` entry in Secrets Manager
- If necessary, restart the `workspace-file-harvester` pod


## Operation

The service runs as a Kubernetes deployment under the `rc` namespace.


### Configuration

The resource catalogue is configured as part of the [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) in the apps/resource-catalogue directory.


### Control

To restart services run `kubectl rollout restart -n rc deployment <service-name>` for Kubernetes cluster or use ArgoCD UI to restart.

Harvesters are produced on schedule - to rerun, set the scheduled time to be in the future and ensure the time has updated in ArgoCD.

To stop services, the service must be removed from ArgoCD configuration.


### Dependencies

There are many dependencies on external data for the resource catalogue, particularly for Planet, which requires the data to be present via the Planet API as and when required by the user. For individual dependencies, check the individual code repositories provided above.


### Backups

All processed files are saved to the `catalogue-population-eodhp` bucket. These can be backed up using the S3 backup procedure if additional backups are required. 


## Development

The resource code is version controlled in the repositories stated above.

New versions should be released by creating a new release using GitHub web UI with a version tag following the pattern v1.2.3. The commit tag will trigger the GitHub action release process.

Alternately, releases may be published directly from the code repository with `make publish version=v1.2.3`, but this should only be used for test releases as the Git commit will not be properly tagged.
