# EODH Repositories

This document is a working inventory of repositories related to the EO Data Hub (EODH). Entries summarise each repository's purpose and, where applicable, note Docker images and ArgoCD locations to help operators and developers find the right code and deployment artifacts quickly.

Notes:

- The "remove" label indicates candidates for archival/removal. This is a prompt for discussion rather than a final decision—please review and confirm before taking action.
- Most descriptions were compiled from repository READMEs and observable configuration (e.g., Docker/ArgoCD); details may be incomplete or outdated. Corrections and updates are welcome.
- Categories are best‑effort and some repos could reasonably appear in multiple sections. Feel free to propose re-categorisations.
- To suggest changes, open a PR or issue against this guide with updated descriptions, links, or categorisation.

## Table of contents

- Backend
  - [configscanning](#configscanning)
  - [eodhp-git-change-scanner](#eodhp-git-change-scanner)
  - [eodhp-spdx-change-scanner](#eodhp-spdx-change-scanner)
  - Accounting
    - [accounting-service](#accounting-service)
    - [billing-collector](#billing-collector)
    - [eodhp-accounting-efs](#eodhp-accounting-efs)
    - [eodhp-accounting-s3-usage](#eodhp-accounting-s3-usage)
    - [eodhp-data-transfer-events](#eodhp-data-transfer-events)
  - Ades
    - [ades-fastapi](#ades-fastapi)
    - [ades-stagein](#ades-stagein)
    - [ades-stageout](#ades-stageout)
    - [calrissian](#calrissian)
    - [eoepca-proc-service-template](#eoepca-proc-service-template)
    - [public-workflow-loader](#public-workflow-loader)
    - [pycalrissian](#pycalrissian)
    - [user-workflows-catalogue-dev](#user-workflows-catalogue-dev)
    - [user-workflows-catalogue-prod](#user-workflows-catalogue-prod)
    - [user-workflows-catalogue-test](#user-workflows-catalogue-test)
    - [workflow-ingester](#workflow-ingester)
    - [workflow-system-test](#workflow-system-test)
    - [zoo-calrissian-runner](#zoo-calrissian-runner)
    - [ZOO-Project](#zoo-project)
  - Api
    - [eodh-openapi](#eodh-openapi)
  - Apis
    - [platform-smoke-tests](#platform-smoke-tests)
  - Auth
    - [eodh-auth-proxy](#eodh-auth-proxy)
    - [eodh-keycloak](#eodh-keycloak)
    - [eodhp-auth-agent](#eodhp-auth-agent)
    - [eodhp-opa-config](#eodhp-opa-config)
    - [keycloakify-starter](#keycloakify-starter)
  - Deployment
    - [eodhp-argocd-deployment](#eodhp-argocd-deployment)
    - [eodhp-deploy-infrastucture](#eodhp-deploy-infrastucture)
    - [eodhp-deploy-supporting-infrastructure](#eodhp-deploy-supporting-infrastructure)
    - [git-change-scanner](#git-change-scanner)
  - Jupyter
    - [eodh-jpyauth](#eodh-jpyauth)
    - [eodh-jupyter-images](#eodh-jupyter-images)
    - [notebooks](#notebooks)
  - Messaging
    - [eodhp-utils](#eodhp-utils)
  - Resource Catalogue
    - [airbus-harvester](#airbus-harvester)
    - [commercial-data-adaptors](#commercial-data-adaptors)
    - [eodh-stac-validator](#eodh-stac-validator)
    - [eodhp-init-catalogs-harvest](#eodhp-init-catalogs-harvest)
    - [eodhp-stac-fastapi](#eodhp-stac-fastapi)
    - [eodhp-stac-fastapi-elasticsearch-opensearch](#eodhp-stac-fastapi-elasticsearch-opensearch)
    - [harvest-transformer](#harvest-transformer)
    - [planet-harvester](#planet-harvester)
    - [planet-stac-converter](#planet-stac-converter)
    - [resource-catalog-support-utils](#resource-catalog-support-utils)
    - [resource-catalogue-fastapi](#resource-catalogue-fastapi)
    - [stac-fastapi-ingester](#stac-fastapi-ingester)
    - [stac-harvester](#stac-harvester)
    - [stac-harvester-configurations](#stac-harvester-configurations)
    - [stac-harvester-ingester](#stac-harvester-ingester)
    - [stac-planet-api](#stac-planet-api)
    - [stac-pydantic](#stac-pydantic)
    - [workspace-catalog-generator](#workspace-catalog-generator)
    - [workspace-file-harvester](#workspace-file-harvester)
  - Titiler
    - [titiler](#titiler)
    - [titiler-stacapi](#titiler-stacapi)

  - Workspaces
    - [eodhp-workspace-controller](#eodhp-workspace-controller)
    - [eodhp-workspace-manager](#eodhp-workspace-manager)
    - [eodhp-workspace-services](#eodhp-workspace-services)
    - [eodhp-workspaces](#eodhp-workspaces)
- Documentation
  - [documentation](#documentation)
  - [eodh-training](#eodh-training)
  - [eodhp-guide](#eodhp-guide)
  - [eodhp-release-notes](#eodhp-release-notes)
  - [eodhp-sprint-reports](#eodhp-sprint-reports)
  - [processes-catalog-api-proposal](#processes-catalog-api-proposal)
- Examples
  - [template-python](#template-python)
  - [eodh-qa-prototype](#eodh-qa-prototype)
  - Ades
    - [ceda-snuggs](#ceda-snuggs)
    - [eodhp-ades-demonstration](#eodhp-ades-demonstration)
    - [example-workflows](#example-workflows)
    - [physrisk-workflow](#physrisk-workflow)
    - [public-workflows](#public-workflows)
    - [tiff-to-cog-workflow](#tiff-to-cog-workflow)
    - [user-workflows](#user-workflows)
    - [workflow-private-data-access](#workflow-private-data-access)
- Frontend
  - [eodhp-rc-ui](#eodhp-rc-ui)
  - [eodhp-web-presence](#eodhp-web-presence)
  - [eodhp-workspace-ui](#eodhp-workspace-ui)
  - [react-starter-app](#react-starter-app)
- Github
  - [.github](#github)
  - [github-actions](#github-actions)
  - [link-checker](#link-checker)
- Other
  - [eodh-eocis-sprint](#eodh-eocis-sprint)
- Project-Management
  - [eodh-qa-pm](#eodh-qa-pm)
  - [eodhp-sparkgeo-pm](#eodhp-sparkgeo-pm)
  - [platform-bugs](#platform-bugs)
  - [project-delivery](#project-delivery)
  - [project_management](#project-management)
  - [user_engagement](#user-engagement)
- Remove
  - [annotations-api](#annotations-api)
  - [annotations-ingester](#annotations-ingester)
  - [annotations-transformer](#annotations-transformer)
  - [commercial-harvester-ingester](#commercial-harvester-ingester)
  - [eodh-demo-client-app](#eodh-demo-client-app)
  - [eodh-pykeycloak](#eodh-pykeycloak)
  - [eodhp-system-tests](#eodhp-system-tests)
  - [generate-annotations-workflow](#generate-annotations-workflow)
  - [harvester-error-notifier](#harvester-error-notifier)
  - [keycloak-offline-token](#keycloak-offline-token)
  - [platform-smoke-tests-tpz](#platform-smoke-tests-tpz)
  - [sparkgeouser-workspace-data](#sparkgeouser-workspace-data)
  - [stac-browser](#stac-browser)
  - [test-catalogue-data](#test-catalogue-data)
  - [tjellicoe-tpzuk-data](#tjellicoe-tpzuk-data)
  - [workflow-harvester](#workflow-harvester)
  - [workspace-logs-aggregator](#workspace-logs-aggregator)
- Tools
  - Clarat
    - [climate-app-docs](#climate-app-docs)
    - [climate-app-lst-workflow](#climate-app-lst-workflow)
    - [climate-app-ui](#climate-app-ui)
  - Eopro
    - [eodh-fe](#eodh-fe)
    - [eodh-fe-infra](#eodh-fe-infra)
    - [eodh-workflows](#eodh-workflows)
  - Integration
    - [eoap-gen](#eoap-gen)
    - [eodh-ac-api](#eodh-ac-api)
    - [eodh-qgis](#eodh-qgis)
    - [pyeodh](#pyeodh)

<a id="github"></a>

### .github

  Storing shared or configuration-related files that affect how GitHub behaves for your organisation

  ![github](https://img.shields.io/badge/github-000000?style=flat)

  ---

<a id="accounting-service"></a>

### accounting-service

  Receives and manages accounting information from around the EODH system via Pulsar, maintains records in PostgreSQL, and serves accounting data to authorized users.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![accounting](https://img.shields.io/badge/accounting-9b59b6?style=flat)

  Docker Image: `public.ecr.aws/eodh/accounting-service`  
  ArgoCD Files: `apps/accounting-service/base/api-deployment.yaml`, `apps/accounting-service/base/ingester-deployment.yaml`

  ---

<a id="ades-fastapi"></a>

### ades-fastapi

  FastAPI-based User Account Service that sits in front of the Workflow Runner service, providing refined interaction with ADES workflows through authenticated requests.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2/ades-fastapi`  
  ArgoCD Files: `apps/ades/base/api/deployment.yaml`

  ---

<a id="ades-stagein"></a>

### ades-stagein

  Stage-in component for ADES to handle input data preparation for workflow execution.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  Docker Image: `public.ecr.aws/eodh/eodhp-ades-stagein`  

  ---

<a id="ades-stageout"></a>

### ades-stageout

  Stage-out component for ADES to handle output data management and transfer after workflow completion.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  Docker Image: `public.ecr.aws/eodh/eodhp-ades-stageout`  

  ---

<a id="airbus-harvester"></a>

### airbus-harvester

  Harvests Earth observation data from Airbus data sources for ingestion into the EODH platform.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2/airbus-harvester`  
  ArgoCD Files: `apps/resource-catalogue/base/airbus/airbus-phr-harvester.yaml`, `apps/resource-catalogue/base/airbus/airbus-pneo-harvester.yaml`, `apps/resource-catalogue/base/airbus/airbus-sar-harvester.yaml`, `apps/resource-catalogue/base/airbus/airbus-spot-harvester.yaml`

  ---

<a id="annotations-api"></a>

### annotations-api

  This API lists and returns the graphs of annotations attached to catalogue entries.

  ![remove](https://img.shields.io/badge/remove-e74c3c?style=flat)

  Docker Image: `public.ecr.aws/eodh/annotations-api`  
  ArgoCD Files: `apps/resource-catalogue/base/annotations/deployment.yaml`

  ---

<a id="annotations-ingester"></a>

### annotations-ingester

  Populates the annotations service with both annotations and a simple DCAT description of catalogue entries. Annotations are small pieces of third-party metadata attached to catalogue entries - ie, they are not produced or maintained by the supplier of the dataset or its core catalogue entry.

  ![remove](https://img.shields.io/badge/remove-e74c3c?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2//ukeodhpannotations-ingester`  
  ArgoCD Files: `apps/resource-catalogue/base/annotations/ingester.yaml`

  ---

<a id="annotations-transformer"></a>

### annotations-transformer

  Receives harvested annotations data in the form provided by users and transforms it to the RDF linked data representation the annotations service needs.

  ![remove](https://img.shields.io/badge/remove-e74c3c?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2/annotations-transformer`  
  ArgoCD Files: `apps/resource-catalogue/base/annotations/transformer.yaml`

  ---

<a id="billing-collector"></a>

### billing-collector

  Tracks CPU and memory usage for each user&#x27;s workspace. Uses Prometheus  to query resource usage and uses Apache Pulsar to send billing events every X seconds.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![accounting](https://img.shields.io/badge/accounting-9b59b6?style=flat)

  Docker Image: `public.ecr.aws/eodh/billing-collector`  
  ArgoCD Files: `apps/billing-collector/base/deployment.yaml`

  ---

<a id="calrissian"></a>

### calrissian

  CWL (Common Workflow Language) implementation designed to run inside Kubernetes clusters for highly efficient and scalable workflow execution.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  Docker Image: `public.ecr.aws/eodh/eodhp-calrissian`  
  ArgoCD Files: `apps/ades/base/values.yaml`

  ---

<a id="ceda-snuggs"></a>

### ceda-snuggs

  Band math CWL workflow

  ![examples](https://img.shields.io/badge/examples-f1c40f?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  Docker Image: `public.ecr.aws/eodh/ceda-snuggs`  

  ---

<a id="climate-app-docs"></a>

### climate-app-docs

  Documentation specific to climate applications and workflows within EODH.

  ![tools](https://img.shields.io/badge/tools-34495e?style=flat)
  ![clarat](https://img.shields.io/badge/clarat-f39c12?style=flat)

  ---

<a id="climate-app-lst-workflow"></a>

### climate-app-lst-workflow

  Land Surface Temperature workflow implementation for climate applications.

  ![tools](https://img.shields.io/badge/tools-34495e?style=flat)
  ![clarat](https://img.shields.io/badge/clarat-f39c12?style=flat)

  ---

<a id="climate-app-ui"></a>

### climate-app-ui

  User interface for climate-related applications and workflows within the EODH platform.

  ![tools](https://img.shields.io/badge/tools-34495e?style=flat)
  ![clarat](https://img.shields.io/badge/clarat-f39c12?style=flat)

  ---

<a id="commercial-data-adaptors"></a>

### commercial-data-adaptors

  Collection of commercial data adaptors. These modules are designed to be run as workflows within the ADES component of the EODH. Each adaptor contains cwl and http scripts to demonstrate deployment and usage in an EODH environment.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  ---

<a id="commercial-harvester-ingester"></a>

### commercial-harvester-ingester

  Service for harvesting and ingesting data from commercial Earth observation data providers.  A planned component that hasn&#x27;t been fully implemented yet

  ![remove](https://img.shields.io/badge/remove-e74c3c?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2//ukeodhpCHANGEME`  

  ---

<a id="configscanning"></a>

### configscanning

  Scans git repositories for changes and can be imported if additional functionality is required if changes are found.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2//ukeodhpconfig-scanning`  

  ---

<a id="documentation"></a>

### documentation

  Contains some of Telespazio&#x27;s documentation about EODHP as a whole (vs an individual component) and working practices.

  ![documentation](https://img.shields.io/badge/documentation-95a5a6?style=flat)

  ---

<a id="eoap-gen"></a>

### eoap-gen

  A CLI tool for generating Earth Observation Application Packages including CWL workflows and Dockerfiles from user supplied python scripts.

  ![tools](https://img.shields.io/badge/tools-34495e?style=flat)
  ![integration](https://img.shields.io/badge/integration-8e44ad?style=flat)

  ---

<a id="eodh-ac-api"></a>

### eodh-ac-api

  API for interacting with Action Creator component and scheduling spatial computation in EO Data Hub.

  ![tools](https://img.shields.io/badge/tools-34495e?style=flat)
  ![integration](https://img.shields.io/badge/integration-8e44ad?style=flat)

  ---

<a id="eodh-auth-proxy"></a>

### eodh-auth-proxy

  A forward auth proxy sidecar to add authentication to outgoing requests for configured domains.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![auth](https://img.shields.io/badge/auth-27ae60?style=flat)

  Docker Image: `public.ecr.aws/eodh/eodh-auth-proxy`  

  ---

<a id="eodh-demo-client-app"></a>

### eodh-demo-client-app

  just a demo repo to trial out app client integrations and debug auth issues.

  ![remove](https://img.shields.io/badge/remove-e74c3c?style=flat)

  ---

<a id="eodh-eocis-sprint"></a>

### eodh-eocis-sprint

  Outputs from the EODH-EOCIS Sprints

  ---

<a id="eodh-fe"></a>

### eodh-fe

  Frontend for EOPro

  ![tools](https://img.shields.io/badge/tools-34495e?style=flat)
  ![eopro](https://img.shields.io/badge/eopro-c0392b?style=flat)

  ---

<a id="eodh-fe-infra"></a>

### eodh-fe-infra

  Infrastructure for EOPro deployment

  ![tools](https://img.shields.io/badge/tools-34495e?style=flat)
  ![eopro](https://img.shields.io/badge/eopro-c0392b?style=flat)

  ---

<a id="eodh-jpyauth"></a>

### eodh-jpyauth

  Custom JupyterHub classes to integrate JupyterHub with the UK EO DataHub.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![jupyter](https://img.shields.io/badge/jupyter-e67e22?style=flat)

  ---

<a id="eodh-jupyter-images"></a>

### eodh-jupyter-images

  A collection of images for the UK EO DataHub Jupyter Hub.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![jupyter](https://img.shields.io/badge/jupyter-e67e22?style=flat)

  ---

<a id="eodh-keycloak"></a>

### eodh-keycloak

  Customized Keycloak setup for EODH

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![auth](https://img.shields.io/badge/auth-27ae60?style=flat)

  Docker Image: `public.ecr.aws/eodh/eodh-keycloak`  
  ArgoCD Files: `apps/keycloak/base/keycloak.yaml`

  ---

<a id="eodh-openapi"></a>

### eodh-openapi

  Merges OpenAPI v3 spec docs from multiple sources and servces them as a single document.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![api](https://img.shields.io/badge/api-1abc9c?style=flat)

  Docker Image: `public.ecr.aws/eodh/eodh-openapi`  
  ArgoCD Files: `apps/docs/base/platform/deployment.yaml`, `apps/docs/base/workspaces/deployment.yaml`

  ---

<a id="eodh-pykeycloak"></a>

### eodh-pykeycloak

  A repository to demonstrate different token endpoint calls.

  ![remove](https://img.shields.io/badge/remove-e74c3c?style=flat)

  ---

<a id="eodh-qa-pm"></a>

### eodh-qa-pm

  Quality assurance project management and testing coordination.

  ![project-management](https://img.shields.io/badge/project-management-7f8c8d?style=flat)

  ---

<a id="eodh-qa-prototype"></a>

### eodh-qa-prototype

  A prototype implementation of the EODH _QA Checker_ concept to illustrate the concept of the QA Checker and its interfaces.

  ![examples](https://img.shields.io/badge/examples-f1c40f?style=flat)
  ![qa](https://img.shields.io/badge/qa-27ae60?style=flat)

  ---

<a id="eodh-qgis"></a>

### eodh-qgis

  QGIS plugin for integrating EODH platform data access within QGIS desktop applications.

  ![tools](https://img.shields.io/badge/tools-34495e?style=flat)
  ![integration](https://img.shields.io/badge/integration-8e44ad?style=flat)

  ---

<a id="eodh-stac-validator"></a>

### eodh-stac-validator

  Validation service for ensuring STAC data conforms to specification standards before ingestion.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  Docker Image: `public.ecr.aws/eodh/eodh-stac-validator`  

  ---

<a id="eodh-training"></a>

### eodh-training

  Training materials, tutorials, and examples for using the EODH platform and APIs.

  ![documentation](https://img.shields.io/badge/documentation-95a5a6?style=flat)

  ---

<a id="eodh-workflows"></a>

### eodh-workflows

  Workflows for Action Creator component on EOPro platform.

  ![tools](https://img.shields.io/badge/tools-34495e?style=flat)
  ![eopro](https://img.shields.io/badge/eopro-c0392b?style=flat)

  ---

<a id="eodhp-accounting-efs"></a>

### eodhp-accounting-efs

  Collects accounting events relating to EFS storage use by workspaces.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![accounting](https://img.shields.io/badge/accounting-9b59b6?style=flat)

  Docker Image: `public.ecr.aws/eodh/accounting-efs`  
  ArgoCD Files: `apps/accounting-service/base/efs-sampler-deployment.yaml`

  ---

<a id="eodhp-accounting-s3-usage"></a>

### eodhp-accounting-s3-usage

  Collects accounting events relating to S3 storage use by workspaces and S3 protocol-based

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![accounting](https://img.shields.io/badge/accounting-9b59b6?style=flat)

  Docker Image: `public.ecr.aws/eodh/accounting-s3-usage`  
  ArgoCD Files: `apps/accounting-service/base/s3-collector-deployment.yaml`

  ---

<a id="eodhp-ades-demonstration"></a>

### eodhp-ades-demonstration

  Example Workflows, Notebooks and HTTP files to Demonstrate the EODH Workflow Runner

  ![examples](https://img.shields.io/badge/examples-f1c40f?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  ---

<a id="eodhp-argocd-deployment"></a>

### eodhp-argocd-deployment

  Deployment repo for the UK EO DataHub Platform. The deployment uses the ArgoCD GitOps framework.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![deployment](https://img.shields.io/badge/deployment-d35400?style=flat)

  ---

<a id="eodhp-auth-agent"></a>

### eodhp-auth-agent

  An API gateway to the EO DataHub to manage authentication and authorization requests.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![auth](https://img.shields.io/badge/auth-27ae60?style=flat)

  Docker Image: `public.ecr.aws/eodh/eodhp-auth-agent`  
  ArgoCD Files: `apps/auth-agent/base/deployment.yaml`, `apps/auth-agent/base/jobs.yaml`

  ---

<a id="eodhp-data-transfer-events"></a>

### eodhp-data-transfer-events

  Collects billing events for CloudFront‐based data transfer by workspaces.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![accounting](https://img.shields.io/badge/accounting-9b59b6?style=flat)

  Docker Image: `public.ecr.aws/eodh/eodhp-accounting-cloudfront`  
  ArgoCD Files: `apps/accounting-service/base/accounting-logs.yaml`

  ---

<a id="eodhp-deploy-infrastucture"></a>

### eodhp-deploy-infrastucture

  The Terraform Infrastructure as Code deployment configuration for the UK EO DataHub Platform.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![deployment](https://img.shields.io/badge/deployment-d35400?style=flat)

  ---

<a id="eodhp-deploy-supporting-infrastructure"></a>

### eodhp-deploy-supporting-infrastructure

  Terraform code for creating and managing supporting infrastructure that serves all EODHP clusters for the UK EO DataHub Platform.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![deployment](https://img.shields.io/badge/deployment-d35400?style=flat)

  ---

<a id="eodhp-git-change-scanner"></a>

### eodhp-git-change-scanner

  Scans Git repositories and harvests new or updated files into the EODHP harvest pipeline.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2/eodhp-git-change-scanner`  
  ArgoCD Files: `apps/ades/base/harvester/sensor.yaml`, `apps/resource-catalogue/base/git-harvester/configuration-harvester.yaml`, `apps/resource-catalogue/base/git-harvester/public-workflows-harvester.yaml`, `apps/resource-catalogue/base/git-harvester/tests/tests.yaml`, `apps/resource-catalogue/base/git-harvester/user-data-harvester.yaml`

  ---

<a id="eodhp-guide"></a>

### eodhp-guide

  Guides to practical aspects of operating and developing the EO DataHub.

  ![documentation](https://img.shields.io/badge/documentation-95a5a6?style=flat)

  ---

<a id="eodhp-init-catalogs-harvest"></a>

### eodhp-init-catalogs-harvest

  Trigger harvesting of the top-level STAC Catalogs for loading into the Resource Catalogue. Currently this includes the `user-datasets` and `supported-datasets` top-level catalogs

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2//ukeodhpeodhp-init-catalog-ingest`  
  ArgoCD Files: `apps/stac-fastapi-2/base/init-catalog-ingest.yaml`

  ---

<a id="eodhp-opa-config"></a>

### eodhp-opa-config

  The OPA policy is deployed by pushing changes to the specified remote branch of this Git repo.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![auth](https://img.shields.io/badge/auth-27ae60?style=flat)

  ---

<a id="eodhp-rc-ui"></a>

### eodhp-rc-ui

  Resource Catalogue User Interface

  ![frontend](https://img.shields.io/badge/frontend-3498db?style=flat)

  ---

<a id="eodhp-release-notes"></a>

### eodhp-release-notes

  Release notes for the work done by Telespazio UK on the EO DataHub platform.

  ![documentation](https://img.shields.io/badge/documentation-95a5a6?style=flat)

  ---

<a id="eodhp-sparkgeo-pm"></a>

### eodhp-sparkgeo-pm

  SparkGeo related issue/project tracking

  ![project-management](https://img.shields.io/badge/project-management-7f8c8d?style=flat)

  ---

<a id="eodhp-spdx-change-scanner"></a>

### eodhp-spdx-change-scanner

  A scanner plugin for configscanning that syncs SPDX license files from a GitHub repo to an S3 bucket.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2/eodhp-spdx-change-scanner`  
  ArgoCD Files: `apps/resource-catalogue/base/spdx/harvester.yaml`

  ---

<a id="eodhp-sprint-reports"></a>

### eodhp-sprint-reports

  Contains reports on each Scrum sprint carried out by Telespazio UK for the EODH project.

  ![documentation](https://img.shields.io/badge/documentation-95a5a6?style=flat)

  ---

<a id="eodhp-stac-fastapi"></a>

### eodhp-stac-fastapi

  Core FastAPI-based implementation of the STAC API for EODH

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  Docker Image: `public.ecr.aws/eodh/eodhp-stac-fastapi`  
  ArgoCD Files: `apps/stac-fastapi-2/base/catalogue-search-service-ingester.yaml`

  ---

<a id="eodhp-stac-fastapi-elasticsearch-opensearch"></a>

### eodhp-stac-fastapi-elasticsearch-opensearch

  Elasticsearch and Opensearch backends for the stac-fastapi project.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  ---

<a id="eodhp-system-tests"></a>

### eodhp-system-tests

  Placeholder tests

  ![remove](https://img.shields.io/badge/remove-e74c3c?style=flat)

  ---

<a id="eodhp-utils"></a>

### eodhp-utils

  Provides shared utilities for EODH services communicating over Pulsar, including message schema definitions, the &quot;Messagers&quot; framework for handling Pulsar event loops and message routing, and an AWS egress classifier for categorizing IP addresses by network region.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![messaging](https://img.shields.io/badge/messaging-2c3e50?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2//ukeodhpeodhp-utils`  

  ---

<a id="eodhp-web-presence"></a>

### eodhp-web-presence

  Public-facing UI using Django/Wagtail for content management, Webpack for frontend assets, and includes account management and public pages (home, documentation, accounts, workspaces)

  ![frontend](https://img.shields.io/badge/frontend-3498db?style=flat)

  Docker Image: `public.ecr.aws/eodh/eodhp-web-presence`  
  ArgoCD Files: `apps/web-presence/base/jobs.yaml`

  ---

<a id="eodhp-workspace-controller"></a>

### eodhp-workspace-controller

  Provides reconcilers for a number of workspace configurations to ensure they remain aligned. This includes reconcilers for namespace, storage (EFS and S3), service, account &amp; AWS IAM Policies

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![workspaces](https://img.shields.io/badge/workspaces-1abc9c?style=flat)

  ---

<a id="eodhp-workspace-manager"></a>

### eodhp-workspace-manager

  Bridges Kubernetes Workspace CRDs and Pulsar: it monitors workspace status changes and publishes them to Pulsar, and applies workspace configuration changes received from Pulsar by updating the Kubernetes Workspace resources.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![workspaces](https://img.shields.io/badge/workspaces-1abc9c?style=flat)

  Docker Image: `public.ecr.aws/eodh/eodhp-workspace-manager`  
  ArgoCD Files: `apps/workspaces/base/manager/deployment.yaml`

  ---

<a id="eodhp-workspace-services"></a>

### eodhp-workspace-services

  An API gateway to the EO DataHub to manage workspace related API requests and process events for workspace management.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![workspaces](https://img.shields.io/badge/workspaces-1abc9c?style=flat)

  Docker Image: `public.ecr.aws/eodh/eodhp-workspace-services`  
  ArgoCD Files: `apps/workspaces/base/api/deployment.yaml`, `apps/workspaces/base/api/jobs.yaml`

  ---

<a id="eodhp-workspace-ui"></a>

### eodhp-workspace-ui

  Frontend React/TypeScript application for managing EODH workspaces, including workspace selection/creation, member management, data loading/publishing, S3 storage, credentials, billing/invoices, catalog browsing, and linked accounts.

  ![frontend](https://img.shields.io/badge/frontend-3498db?style=flat)

  ---

<a id="eodhp-workspaces"></a>

### eodhp-workspaces

  Manages the configuration of the UK EO DataHub Platform workspaces.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![workspaces](https://img.shields.io/badge/workspaces-1abc9c?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2//ukeodhpeodhp-workspaces`  

  ---

<a id="eoepca-proc-service-template"></a>

### eoepca-proc-service-template

  This repository is cloned by the Workflow Runner and used to deploy new workflows to the ADES, it defines pre- and post-processing steps when executing workflows.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  ---

<a id="example-workflows"></a>

### example-workflows

  A number of workflows, defined as Common Workflow Language (CWL) scripts

  ![examples](https://img.shields.io/badge/examples-f1c40f?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  ---

<a id="generate-annotations-workflow"></a>

### generate-annotations-workflow

  Used as a testing tool for the annotations ingest pipeline. Development has been suspended on the annotations pipeline and this is no longer maintained.

  ![remove](https://img.shields.io/badge/remove-e74c3c?style=flat)

  ---

<a id="git-change-scanner"></a>

### git-change-scanner

  Automates GitOps-style deployment for the AIPIPE machine learning platform

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![deployment](https://img.shields.io/badge/deployment-d35400?style=flat)

  ---

<a id="github-actions"></a>

### github-actions

  Contains workflows for GitHub actions used by the EO-DataHub

  ![github](https://img.shields.io/badge/github-000000?style=flat)

  ---

<a id="harvest-transformer"></a>

### harvest-transformer

  A service for transforming harvested STAC metadata to make it suitable for the EODH catalogue.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  Docker Image: `public.ecr.aws/eodh/harvest-transformer`  
  ArgoCD Files: `apps/resource-catalogue/base/git-harvester/configuration-transformer.yaml`, `apps/resource-catalogue/base/transformer.yaml`

  ---

<a id="harvester-error-notifier"></a>

### harvester-error-notifier

  Template repository (not fully implemented) for notifying on harvester errors.

  ![remove](https://img.shields.io/badge/remove-e74c3c?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2/harvester-error-notifier`  

  ---

<a id="keycloak-offline-token"></a>

### keycloak-offline-token

  Script to demonstrate how application developers can get a Keycloak offline refresh token and then use this to request an access token for use with API calls

  ![remove](https://img.shields.io/badge/remove-e74c3c?style=flat)

  ---

<a id="keycloakify-starter"></a>

### keycloakify-starter

  Enables building custom Keycloak login/auth themes with React/TypeScript instead of FreeMarker templates.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![auth](https://img.shields.io/badge/auth-27ae60?style=flat)

  ---

<a id="link-checker"></a>

### link-checker

  Standalone GitHub Action and CLI tool that checks websites for broken links using linkinator.

  ![github](https://img.shields.io/badge/github-000000?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2//ukeodhpCHANGEME`  

  ---

<a id="notebooks"></a>

### notebooks

  Official Jupyter Docker Stacks repository: ready-to-run Docker images for Jupyter applications and interactive computing tools.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![jupyter](https://img.shields.io/badge/jupyter-e67e22?style=flat)

  ---

<a id="physrisk-workflow"></a>

### physrisk-workflow

  A command-line interface tool for running Physrisk as a workflow, designed to assess physical climate risks for assets.

  ![examples](https://img.shields.io/badge/examples-f1c40f?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  ---

<a id="planet-harvester"></a>

### planet-harvester

  Harvests collections from the Planet catalogue. Planet is large and contains many items - items need to be collected via the API

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2/planet-harvester`  
  ArgoCD Files: `apps/resource-catalogue/base/planet/harvester.yaml`

  ---

<a id="planet-stac-converter"></a>

### planet-stac-converter

  Tool that converts STAC query requests into Planet Labs API calls, enabling STAC-standard searches on Planet&#x27;s satellite imagery catalog.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  ---

<a id="platform-bugs"></a>

### platform-bugs

  This is a place for partners to report bugs and feature requests against the hub platform.

  ![project-management](https://img.shields.io/badge/project-management-7f8c8d?style=flat)

  ---

<a id="platform-smoke-tests"></a>

### platform-smoke-tests

  Provides an easy way to smoke test EODH APIs. All tests are automatically generated from available OpenAPI specifications.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![apis](https://img.shields.io/badge/apis-2ecc71?style=flat)

  ---

<a id="platform-smoke-tests-tpz"></a>

### platform-smoke-tests-tpz

  This was a temporary repo set up to create a PR for platform-smoke-tests repo

  ![remove](https://img.shields.io/badge/remove-e74c3c?style=flat)

  ---

<a id="processes-catalog-api-proposal"></a>

### processes-catalog-api-proposal

  Proposed structures and documents for the EODH (Earth Observation Data Hub) process catalogue, which enables discovery and execution of Earth observation workflows and notebooks

  ![documentation](https://img.shields.io/badge/documentation-95a5a6?style=flat)

  ---

<a id="project-delivery"></a>

### project-delivery

  Project delivery issues and tracking.

  ![project-management](https://img.shields.io/badge/project-management-7f8c8d?style=flat)

  ---

<a id="project-management"></a>

### project_management

  Project management issues, and tracking for platform development.

  ![project-management](https://img.shields.io/badge/project-management-7f8c8d?style=flat)

  ---

<a id="public-workflow-loader"></a>

### public-workflow-loader

  Provides a command line application that can deploy any workflows in a given directory to the Workflow Runner on the EO DataHub Platform.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  Docker Image: `public.ecr.aws/eodh/workflow-loader`  
  ArgoCD Files: `apps/ades/base/default-loader/job.yaml`

  ---

<a id="public-workflows"></a>

### public-workflows

  Publicly available workflows that can be executed by EODH platform users.

  ![examples](https://img.shields.io/badge/examples-f1c40f?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  ---

<a id="pycalrissian"></a>

### pycalrissian

  Python library for interacting with Calrissian workflow execution engine.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  ---

<a id="pyeodh"></a>

### pyeodh

  A lightweight Python client for easy access to EODH APIs.

  ![tools](https://img.shields.io/badge/tools-34495e?style=flat)
  ![integration](https://img.shields.io/badge/integration-8e44ad?style=flat)

  ---

<a id="react-starter-app"></a>

### react-starter-app

  Template providing a minimal setup to get React working in Vite with HMR and some ESLint rules.

  ![frontend](https://img.shields.io/badge/frontend-3498db?style=flat)

  ---

<a id="resource-catalog-support-utils"></a>

### resource-catalog-support-utils

  A FastAPI web service that generates QLR (QGIS Layer) files from STAC items with pre-constructed XYZ tile service URL

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  ---

<a id="resource-catalogue-fastapi"></a>

### resource-catalogue-fastapi

  An API to allow interaction with Stac-FastAPI within the EO Data Hub

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  Docker Image: `public.ecr.aws/eodh/resource-catalogue-fastapi`  
  ArgoCD Files: `apps/resource-catalogue/base/workspaces/manage-catalogue-api.yaml`

  ---

<a id="sparkgeouser-workspace-data"></a>

### sparkgeouser-workspace-data

  Repository for workspace-specific user datasets and access policies for the sparkgeouser user

  ![remove](https://img.shields.io/badge/remove-e74c3c?style=flat)

  ---

<a id="stac-browser"></a>

### stac-browser

  STAC browser for static catalogs

  ![remove](https://img.shields.io/badge/remove-e74c3c?style=flat)

  ---

<a id="stac-fastapi-ingester"></a>

### stac-fastapi-ingester

  An ingester to load STAC Catalogs, Collections and Items into STAC-Fastapi, read from S3 bucket, triggered by Pulsar messages.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2/eodhp-stac-fastapi-ingester`  
  ArgoCD Files: `apps/stac-fastapi-2/base/catalogue-search-service-ingester.yaml`

  ---

<a id="stac-harvester"></a>

### stac-harvester

  Harvests STAC catalogs from upstream sources. It collects STAC records from source catalogs, stores them in S3, and publishes Pulsar messages for added, changed, and deleted entries by comparing with stored hashes.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2//ukeodhpstac-harvester`  

  ---

<a id="stac-harvester-configurations"></a>

### stac-harvester-configurations

  Contains the configurations for harvesting STAC catalogues

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  ---

<a id="stac-harvester-ingester"></a>

### stac-harvester-ingester

  Takes catalogue configuration data from a pulsar message and creates a STAC Harvester for
each catalogue.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2//ukeodhpstac-harvester-ingester`  
  ArgoCD Files: `apps/resource-catalogue/base/git-harvester/configuration-ingester.yaml`

  ---

<a id="stac-planet-api"></a>

### stac-planet-api

  Converter from STAC request to Planet request then from Planet response to STAC response.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  Docker Image: `public.ecr.aws/eodh/stac-planet-api`  
  ArgoCD Files: `apps/resource-catalogue/base/planet/deployment.yaml`

  ---

<a id="stac-pydantic"></a>

### stac-pydantic

  Pydantic models for STAC Catalogs, Collections, Items, and the STAC API

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  ---

<a id="template-python"></a>

### template-python

  A template repository for new UKEODHP Python-base software components.

  ![examples](https://img.shields.io/badge/examples-f1c40f?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2//ukeodhpCHANGEME`  

  ---

<a id="test-catalogue-data"></a>

### test-catalogue-data

  A data repository for testing and experimentation. Assume that any and all branches may change.

  ![remove](https://img.shields.io/badge/remove-e74c3c?style=flat)

  ---

<a id="tiff-to-cog-workflow"></a>

### tiff-to-cog-workflow

  A CWL workflow for converting a remote TIFF file to a COG and generating a STAC catalog

  ![examples](https://img.shields.io/badge/examples-f1c40f?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  ---

<a id="titiler"></a>

### titiler

  A modern dynamic tile server built on top of FastAPI and Rasterio/GDAL

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![titiler](https://img.shields.io/badge/titiler-16a085?style=flat)

  Docker Image: `public.ecr.aws/eodh/titiler-core`  
  ArgoCD Files: `apps/titiler/base/titiler/deployment.yaml`

  ---

<a id="titiler-stacapi"></a>

### titiler-stacapi

  Connect titiler to STAC APIs

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![titiler](https://img.shields.io/badge/titiler-16a085?style=flat)

  Docker Image: `public.ecr.aws/eodh/titiler-stacapi`  
  ArgoCD Files: `apps/titiler/base/stac-titiler/deployment.yaml`

  ---

<a id="tjellicoe-tpzuk-data"></a>

### tjellicoe-tpzuk-data

  Repository for workspace-specific user datasets and access policies for the particular user

  ![remove](https://img.shields.io/badge/remove-e74c3c?style=flat)

  ---

<a id="user-workflows"></a>

### user-workflows

  Contains user specific processing workflows for the EO Data Hub.

  ![examples](https://img.shields.io/badge/examples-f1c40f?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  ---

<a id="user-workflows-catalogue-dev"></a>

### user-workflows-catalogue-dev

  Allows users to deploy workflows and access policies directly from their CWL and JSON definitions

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  ---

<a id="user-workflows-catalogue-prod"></a>

### user-workflows-catalogue-prod

  Allows users to deploy workflows and access policies directly from their CWL and JSON definitions

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  ---

<a id="user-workflows-catalogue-test"></a>

### user-workflows-catalogue-test

  Allows users to deploy workflows and access policies directly from their CWL and JSON definitions

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  ---

<a id="user-engagement"></a>

### user_engagement

  Manage user engagement tasks

  ![project-management](https://img.shields.io/badge/project-management-7f8c8d?style=flat)

  ---

<a id="workflow-harvester"></a>

### workflow-harvester

  Empty repo

  ![remove](https://img.shields.io/badge/remove-e74c3c?style=flat)

  ---

<a id="workflow-ingester"></a>

### workflow-ingester

  Defines an image that listens to a Pulsar topic and upon receiving a message updates the specified files in S3 and then sends requests to the ADES in order to Deploy, Update or Delete the workflow from the specified user&#x27;s workspace.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2/eodhp-workflow-ingester`  
  ArgoCD Files: `apps/ades/base/ingester/deployment.yaml`

  ---

<a id="workflow-private-data-access"></a>

### workflow-private-data-access

  An example Docker image and Common Workflow Language (CWL) Application Package that can be used with the EO DataHub Workflow Runner to demonstrate accessing private data within workflow steps

  ![examples](https://img.shields.io/badge/examples-f1c40f?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  ---

<a id="workflow-system-test"></a>

### workflow-system-test

  Provides a full end-to-end systems test of the workflow system included in EODHP

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2/workflow-system-test`  
  ArgoCD Files: `apps/ades/base/tests/test-file.yaml`, `apps/ades/base/tests/test-url.yaml`

  ---

<a id="workspace-catalog-generator"></a>

### workspace-catalog-generator

  Contains code to generate required STAC catalogs for EODHP workspaces.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2//ukeodhpeodhp-workspace-catalog-generator`  
  ArgoCD Files: `apps/resource-catalogue/base/workspaces/workspaces-harvester.yaml`

  ---

<a id="workspace-file-harvester"></a>

### workspace-file-harvester

  Collects user-uploaded files from S3. STAC files are passed to the transformer and ingestor for harvesting and access policy files are used to update public access of files, folders and workflows.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![rc](https://img.shields.io/badge/rc-9b59b6?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2/workspace-file-harvester`  
  ArgoCD Files: `apps/resource-catalogue/base/workspaces/deployment.yaml`

  ---

<a id="workspace-logs-aggregator"></a>

### workspace-logs-aggregator

  Template repository (not fully implemented) intended to aggregate logs from EODH workspaces

  ![remove](https://img.shields.io/badge/remove-e74c3c?style=flat)

  Docker Image: `public.ecr.aws/n1b3o1k2//ukeodhpCHANGEME`  

  ---

<a id="zoo-calrissian-runner"></a>

### zoo-calrissian-runner

  Python library for bridging zoo execution context and calrissian

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  ---

<a id="zoo-project"></a>

### ZOO-Project

  Implements OGC standards as a polyglot server, enabling execution of processing services across multiple programming languages.

  ![backend](https://img.shields.io/badge/backend-34495e?style=flat)
  ![ades](https://img.shields.io/badge/ades-2980b9?style=flat)

  ---
