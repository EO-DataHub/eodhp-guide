# Data Adaptors

## Summary

Data adaptors are used to order data from Airbus and Planet.


### Code Repositories and Artifacts

- Deployment is configured in https://github.com/EO-DataHub/eodhp-argocd-deployment repository, apps/resource-catalogue directory
- Code available in https://github.com/EO-DataHub/commercial-data-adaptors repository

#### Airbus
- Container image published to public.ecr.aws/eodh/airbus-optical-adaptor and public.ecr.aws/eodh/airbus-sar-adaptor AWS ECR

#### Planet
- Container image published to public.ecr.aws/eodh/planet-adaptor AWS ECR


### Dependent Services

[//]: # (The elasticsearch ingester takes inputs from the harvest transformer.)


## Operation

[//]: # (The service runs as a Kubernetes deployment under the `rc` namespace.)


### Configuration

[//]: # (The resource catalogue is configured as part of the [ArgoCD deployment repo]&#40;https://github.com/EO-DataHub/eodhp-argocd-deployment&#41; in the apps/resource-catalogue directory.)


### Control

[//]: # (To restart services run `kubectl rollout restart -n rc deployment <service-name>` for Kubernetes cluster or use ArgoCD UI to restart.)

[//]: # ()
[//]: # (Harvesters are produced on schedule - to rerun, set the scheduled time to be in the future and ensure the time has updated in ArgoCD.)

[//]: # ()
[//]: # (To stop services, the service must be removed from ArgoCD configuration.)


### Dependencies

[//]: # (There are many dependencies on external data for the resource catalogue, particularly for Planet, which requires the data to be present via the Planet API as and when required by the user. For individual dependencies, check the individual code repositories provided above.)


### Backups

[//]: # (All processed files are saved to the `catalogue-population-eodhp` bucket. These can be backed up using the S3 backup procedure if additional backups are required. )


## Development

[//]: # (The resource code is version controlled in the repositories stated above.)

[//]: # ()
[//]: # (New versions should be released by creating a new release using GitHub web UI with a version tag following the pattern v1.2.3. The commit tag will trigger the GitHub action release process.)

[//]: # ()
[//]: # (Alternately, releases may be published directly from the code repository with `make publish version=v1.2.3`, but this should only be used for test releases as the Git commit will not be properly tagged.)
