# EODHP Platform Components

## Resource Catalogue

### Lead Developer

- Tom Jellicoe
- Mark Small (UI)

### Repositories

Configuration:

- catalogue-data
- catalogue-supported-data (Hannah Collingwood)
- element84-data (Hannah Collingwood)
- stac-harvester-configurations (Hannah Collingwood)
- test-catalogue-data
- user-data-catalogue

Code:

- annotations-ingester (Alex Hayward)
- annotations-service (Alex Hayward)
- annotations-transformer
- catalogue-search-service
- commercial-data-adaptors
- configscanning (Hannah Collingwood)
- eodhp-git-change-scanner (Hannah Collingwood)
- git-change-scanner (OBSOLETE) (Alex Hayward)
- eodhp-utils (Hannah Collingwood)
- eodhp-workflow-transformer
- harvest-transformer
- harvest-error-notifier (Hannah Collingwood)
- resource-catalogue-fastapi (Alex Palmer)
- stac-fastapi
- stac-fastapi-elasticsearch-opensearch
- stac-fastapi-ingester
- stac-harvester (Hannah Collingwood)
- stac-harvester-ingester (Hannah Collingwood)
- workspace-catalog-generator
- UKEODHP/catalogue-search-service-temp
- UKEODHP/eodhp-init-catalogs-harvest
- UKEODHP/stac-fastapi-data-loader

## Workflow and Analysis System

### Lead Developer

- Jonny Langstone

### Repositories

- eodhp-workspace-controller
- eodhp-workspace-manager
- eodhp-workspace-services
- UKEODHP/apphub-singleuser
- UKEODHP/workspace-controller-testenv

## Workflow Runner

### Lead Developer

- Tom Jellicoe

### Repositories

Code:

- ades-fastapi
- ades-workflow-examples
- eodhp-ades-demonstration
- eodhp-ades-workspace-access
- generate-annotations-workflow
- eoepca-proc-service-template
- stac-workflow-generator
- test-workflow-store
- workflow-ingester
- UKEODHP/ades-auth-proxy (Steven Gillies)
- UKEODHP/ades-stagein
- UKEODHP/ades-stageout
- UKEODHP/calrissian
- UKEODHP/pycalrissian
- UKEODHP/zoo-calrissian-runner
- UKEODHP/ZOO-Project

Configuration:

- public-workflows
- user-workflows-catalogue-dev

## Data Access Services

### Lead Developer

- James Hinton

### Repositories

Code:

- eodhp-convert-netcdf
- titiler-stacapi
- titiler-xarray
- UKEODHP/eodhp-eoxserver (OBSOLETE)
- UKEODHP/eodhp-eoxserver-climate-data (OBSOLETE)
- UKEODHP/eodhp-eoxserver-django (OBSOLETE)
- UKEODHP/eodhp-eoxserver-netcdf-demo (OBSOLETE)
- UKEODHP/eodhp-eoxviewserver (OBSOLETE)
- UKEODHP/eoxserver (OBSOLETE)
- UKEODHP/eoxviewserver-demo (OBSOLETE)
- UKEODHP/eoxvs-register-stac-items (OBSOLETE)
- UKEODHP/titiler-stacapi

## Data Streams

### Lead Developer

- Alex Palmer

### Repositories

Code:

- airbus-harvester
- planet-harvester (Hannah Collingwood)

## Web Presence

### Lead Developer

- Hannah Collingwood
- Mark Small (UI)

### Repositories

Code:

- eodhp-web-presence
- eodhp-workspace-ui (James Hinton)
- react-starter-app (James Hinton)

Configuration:

- eodhp-web-presence-helm

## Catalogue UI

### Lead Developer

- James Hinton

### Repositories

Code:

- eodhp-resource-catalogue-ui
- stac-browser

## IAM

### Lead Developer

- Steven Gillies
- Jonny Langstone

### Repositories

Code:

- eodh-demo-client-app
- eodhp-auth-agent
- keycloak-offline-token

Configuration:

- eodhp-opa-config

## Event Notification Service

### Lead Developer

- Hannah Collingwood

## System Management

### Lead Developer

- Steven Gillies

### Repositories

Configuration:

- eodhp-argocd-deployment
- eodhp-deploy-infrastucture
- eodhp-deploy-supporting-infrastructure
- eodhp-workspaces
- UKEODHP/argocd-autopilot-reference-deployment (OBSOLETE)
- UKEODHP/eck-stack
- UKEODHP/linkerd-demo
- UKEODHP/pulsar-demo (OBSOLETE)

Code:

- github-actions (Hannah Collingwood)

## Testing

### Lead Developer

- Alex Palmer

### Repositories

Code:

- resource-catalogue-tests
- workflow-system-test (Tom Jellicoe)
- UKEODHP/argo-workflow-system-tests
- UKEODHP/public-workflows-test (Tom Jellicoe)
- UKEODHP/resource-catalogue-tests (Hannah Collingwood)

## Other

### Repositories

Documentation:

- documentation (Alex Hayward)
- eodhp-guide (Steven Gillies)
- eodhp-sprint-reports (Steven Gillies)
- UKEODHP/eodhp-system-tests-tradeoff (OBSOLETE)
- UKEODHP/system-test-tradeoff (OBSOLETE)
- UKEODHP/template-python (Alex Hayward)

# Lead Responsibilities

A lead might not do all (or even most) development on a component but should help plan the
functional and architectural evolution of the component - developing or writing user stories,
for example.

A lead is also responsible for the repositories for that component unless otherwise listed.
Except for those marked OBSOLETE, the person responsible should:

- Watch the repository for at least security alerts and respond to them.
- Keep dependencies up-to-date or keeping it up-to-date with an upstream repo it was forked from.
- Keep the build system running and up-to-date.
- Do any other maintenance of the repo that isn't a part of particular stories.
