---
title: EODH Repositories
doc_status: needs-verification
last_reviewed: 2026-06-29
reviewed_by: recmanj
review_notes:
tags:
  - gitops
---
# EODH Repositories

This page is a working inventory of the repositories in the [EO-DataHub GitHub organisation](https://github.com/EO-DataHub). Each table gives a repository's purpose and, where applicable, its Docker image and ArgoCD location, so operators and developers can find the right code and deployment artifacts quickly. Use the page contents to jump to a section.

Notes:

- Synced against the EO-DataHub GitHub organisation on 2026-10-26.
- Repositories are public unless marked 🔒 (private). Archived repositories are read-only on GitHub and are listed in the [Archived](#archived) section.
- ArgoCD paths are relative to the root of `eodhp-argocd-deployment`. Where a repository is deployed by several files in one directory, the directory is shown.
- Most descriptions were compiled from repository READMEs and observable configuration (e.g., Docker/ArgoCD); details may be incomplete or outdated. Corrections and updates are welcome.
- Categories are best‑effort and some repos could reasonably appear in multiple sections. Feel free to propose re-categorisations.
- To suggest changes, open a PR or issue against this guide with updated descriptions, links, or categorisation.

## Backend

### Config scanning

| Repository | Purpose | Docker image | ArgoCD |
|---|---|---|---|
| [configscanning](https://github.com/EO-DataHub/configscanning) | Scans git repositories for changes and can be imported if additional functionality is required if changes are found. | `public.ecr.aws/n1b3o1k2//ukeodhpconfig-scanning` |  |
| [eodhp-git-change-scanner](https://github.com/EO-DataHub/eodhp-git-change-scanner) | Scans Git repositories and harvests new or updated files into the EODHP harvest pipeline. | `public.ecr.aws/n1b3o1k2/eodhp-git-change-scanner` | `apps/ades/base/harvester/sensor.yaml`<br>`apps/resource-catalogue/base/git-harvester/configuration-harvester.yaml`<br>`apps/resource-catalogue/base/git-harvester/public-workflows-harvester.yaml`<br>`apps/resource-catalogue/base/git-harvester/tests/tests.yaml`<br>`apps/resource-catalogue/base/git-harvester/user-data-harvester.yaml` |
| [eodhp-spdx-change-scanner](https://github.com/EO-DataHub/eodhp-spdx-change-scanner) | A scanner plugin for configscanning that syncs SPDX license files from a GitHub repo to an S3 bucket. | `public.ecr.aws/n1b3o1k2/eodhp-spdx-change-scanner` | `apps/resource-catalogue/base/spdx/harvester.yaml` |

### Accounting

| Repository | Purpose | Docker image | ArgoCD |
|---|---|---|---|
| [accounting-service](https://github.com/EO-DataHub/accounting-service) | Receives and manages accounting information from around the EODH system via Pulsar, maintains records in PostgreSQL, and serves accounting data to authorized users. | `public.ecr.aws/eodh/accounting-service` | `apps/accounting-service/base/api-deployment.yaml`<br>`apps/accounting-service/base/ingester-deployment.yaml` |
| [billing-collector](https://github.com/EO-DataHub/billing-collector) | Tracks CPU and memory usage for each user's workspace. Uses Prometheus to query resource usage and uses Apache Pulsar to send billing events every X seconds. | `public.ecr.aws/eodh/billing-collector` | `apps/billing-collector/base/deployment.yaml` |
| [eodhp-accounting-efs](https://github.com/EO-DataHub/eodhp-accounting-efs) | Collects accounting events relating to EFS storage use by workspaces. | `public.ecr.aws/eodh/accounting-efs` | `apps/accounting-service/base/efs-sampler-deployment.yaml` |
| [eodhp-accounting-s3-usage](https://github.com/EO-DataHub/eodhp-accounting-s3-usage) | Collects accounting events relating to S3 storage use by workspaces and S3 protocol-based access to those stores. HTTPS access is collected by eodhp-data-transfer-events. | `public.ecr.aws/eodh/accounting-s3-usage` | `apps/accounting-service/base/s3-collector-deployment.yaml` |
| [eodhp-data-transfer-events](https://github.com/EO-DataHub/eodhp-data-transfer-events) | Collects billing events for CloudFront‐based data transfer by workspaces. | `public.ecr.aws/eodh/eodhp-accounting-cloudfront` | `apps/accounting-service/base/accounting-logs.yaml` |

### ADES

| Repository | Purpose | Docker image | ArgoCD |
|---|---|---|---|
| [ades-fastapi](https://github.com/EO-DataHub/ades-fastapi) | FastAPI-based User Account Service that sits in front of the Workflow Runner service, providing refined interaction with ADES workflows through authenticated requests. | `public.ecr.aws/n1b3o1k2/ades-fastapi` | `apps/ades/base/api/deployment.yaml` |
| [ades-stagein](https://github.com/EO-DataHub/ades-stagein) | Stage-in component for ADES to handle input data preparation for workflow execution. | `public.ecr.aws/eodh/eodhp-ades-stagein` |  |
| [ades-stageout](https://github.com/EO-DataHub/ades-stageout) | Stage-out component for ADES to handle output data management and transfer after workflow completion. | `public.ecr.aws/eodh/eodhp-ades-stageout` |  |
| [calrissian](https://github.com/EO-DataHub/calrissian) | CWL (Common Workflow Language) implementation designed to run inside Kubernetes clusters for highly efficient and scalable workflow execution. | `public.ecr.aws/eodh/eodhp-calrissian` | `apps/ades/base/values.yaml` |
| [eoepca-proc-service-template](https://github.com/EO-DataHub/eoepca-proc-service-template) | This repository is cloned by the Workflow Runner and used to deploy new workflows to the ADES, it defines pre- and post-processing steps when executing workflows. |  |  |
| [public-workflow-loader](https://github.com/EO-DataHub/public-workflow-loader) | Provides a command line application that can deploy any workflows in a given directory to the Workflow Runner on the EO DataHub Platform. | `public.ecr.aws/eodh/workflow-loader` | `apps/ades/base/default-loader/job.yaml` |
| [pycalrissian](https://github.com/EO-DataHub/pycalrissian) | Python library for interacting with Calrissian workflow execution engine. |  |  |
| [user-workflows-catalogue-dev](https://github.com/EO-DataHub/user-workflows-catalogue-dev) 🔒 | Allows users to deploy workflows and access policies directly from their CWL and JSON definitions |  |  |
| [user-workflows-catalogue-prod](https://github.com/EO-DataHub/user-workflows-catalogue-prod) 🔒 | Allows users to deploy workflows and access policies directly from their CWL and JSON definitions |  |  |
| [user-workflows-catalogue-test](https://github.com/EO-DataHub/user-workflows-catalogue-test) 🔒 | Allows users to deploy workflows and access policies directly from their CWL and JSON definitions |  |  |
| [workflow-ingester](https://github.com/EO-DataHub/workflow-ingester) | Defines an image that listens to a Pulsar topic and upon receiving a message updates the specified files in S3 and then sends requests to the ADES in order to Deploy, Update or Delete the workflow from the specified user's workspace. | `public.ecr.aws/n1b3o1k2/eodhp-workflow-ingester` | `apps/ades/base/ingester/deployment.yaml` |
| [workflow-system-test](https://github.com/EO-DataHub/workflow-system-test) | Provides a full end-to-end systems test of the workflow system included in EODHP | `public.ecr.aws/n1b3o1k2/workflow-system-test` | `apps/ades/base/tests/` |
| [zoo-calrissian-runner](https://github.com/EO-DataHub/zoo-calrissian-runner) | Python library for bridging zoo execution context and calrissian |  |  |
| [ZOO-Project](https://github.com/EO-DataHub/ZOO-Project) | Implements OGC standards as a polyglot server, enabling execution of processing services across multiple programming languages. |  |  |

### Annotations

| Repository | Purpose | Docker image | ArgoCD |
|---|---|---|---|
| [annotations-api](https://github.com/EO-DataHub/annotations-api) | This API lists and returns the graphs of annotations attached to catalogue entries. | `public.ecr.aws/eodh/annotations-api` | `apps/resource-catalogue/base/annotations/deployment.yaml` |
| [annotations-ingester](https://github.com/EO-DataHub/annotations-ingester) | Populates the annotations service with both annotations and a simple DCAT description of catalogue entries. Annotations are small pieces of third-party metadata attached to catalogue entries - ie, they are not produced or maintained by the supplier of the dataset or its core catalogue entry. | `public.ecr.aws/n1b3o1k2//ukeodhpannotations-ingester` | `apps/resource-catalogue/base/annotations/ingester.yaml` |
| [annotations-transformer](https://github.com/EO-DataHub/annotations-transformer) | Receives harvested annotations data in the form provided by users and transforms it to the RDF linked data representation the annotations service needs. | `public.ecr.aws/n1b3o1k2/annotations-transformer` | `apps/resource-catalogue/base/annotations/transformer.yaml` |

### APIs

| Repository | Purpose | Docker image | ArgoCD |
|---|---|---|---|
| [eodh-openapi](https://github.com/EO-DataHub/eodh-openapi) | Merges OpenAPI v3 spec docs from multiple sources and serves them as a single document. | `public.ecr.aws/eodh/eodh-openapi` | `apps/docs/base/platform/deployment.yaml`<br>`apps/docs/base/workspaces/deployment.yaml` |
| [platform-smoke-tests](https://github.com/EO-DataHub/platform-smoke-tests) 🔒 | Provides an easy way to smoke test EODH APIs. All tests are automatically generated from available OpenAPI specifications. |  |  |

### Auth

| Repository | Purpose | Docker image | ArgoCD |
|---|---|---|---|
| [eodh-keycloak](https://github.com/EO-DataHub/eodh-keycloak) | Customized Keycloak setup for EODH | `public.ecr.aws/eodh/eodh-keycloak` | `apps/keycloak/base/keycloak.yaml` |
| [eodhp-auth-agent](https://github.com/EO-DataHub/eodhp-auth-agent) | An API gateway to the EO DataHub to manage authentication and authorization requests. | `public.ecr.aws/eodh/eodhp-auth-agent` | `apps/auth-agent/base/deployment.yaml`<br>`apps/auth-agent/base/jobs.yaml` |
| [eodhp-opa-config](https://github.com/EO-DataHub/eodhp-opa-config) 🔒 | The OPA policy is deployed by pushing changes to the specified remote branch of this Git repo. |  |  |
| [keycloakify-starter](https://github.com/EO-DataHub/keycloakify-starter) | Enables building custom Keycloak login/auth themes with React/TypeScript instead of FreeMarker templates. |  |  |

### Deployment

| Repository | Purpose |
|---|---|
| [eodhp-argocd-deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment) 🔒 | Deployment repo for the UK EO DataHub Platform. The deployment uses the ArgoCD GitOps framework. |
| [eodhp-deploy-infrastucture](https://github.com/EO-DataHub/eodhp-deploy-infrastucture) 🔒 | The Terraform Infrastructure as Code deployment configuration for the UK EO DataHub Platform. |
| [eodhp-deploy-supporting-infrastructure](https://github.com/EO-DataHub/eodhp-deploy-supporting-infrastructure) 🔒 | Terraform code for creating and managing supporting infrastructure that serves all EODHP clusters for the UK EO DataHub Platform. |

### Jupyter

| Repository | Purpose |
|---|---|
| [eodh-jpyauth](https://github.com/EO-DataHub/eodh-jpyauth) | Custom JupyterHub classes to integrate JupyterHub with the UK EO DataHub. |
| [eodh-jupyter-images](https://github.com/EO-DataHub/eodh-jupyter-images) | A collection of images for the UK EO DataHub Jupyter Hub. |
| [notebooks](https://github.com/EO-DataHub/notebooks) | Official Jupyter Docker Stacks repository: ready-to-run Docker images for Jupyter applications and interactive computing tools. |

### Messaging

| Repository | Purpose | Docker image |
|---|---|---|
| [eodhp-utils](https://github.com/EO-DataHub/eodhp-utils) | Provides shared utilities for EODH services communicating over Pulsar, including message schema definitions, the "Messagers" framework for handling Pulsar event loops and message routing, and an AWS egress classifier for categorizing IP addresses by network region. | `public.ecr.aws/n1b3o1k2//ukeodhpeodhp-utils` |

### Resource Catalogue

| Repository | Purpose | Docker image | ArgoCD |
|---|---|---|---|
| [airbus-harvester](https://github.com/EO-DataHub/airbus-harvester) | Harvests Earth observation data from Airbus data sources for ingestion into the EODH platform. | `public.ecr.aws/n1b3o1k2/airbus-harvester` | `apps/resource-catalogue/base/airbus/` |
| [commercial-data-adaptors](https://github.com/EO-DataHub/commercial-data-adaptors) | Collection of commercial data adaptors. These modules are designed to be run as workflows within the ADES component of the EODH. Each adaptor contains cwl and http scripts to demonstrate deployment and usage in an EODH environment. |  |  |
| [element84-harvester](https://github.com/EO-DataHub/element84-harvester) | Harvests the Element84 Earth Search STAC catalogue (`sentinel-2-c1-l2a` collection only) into the EODH `public` catalogue tree. Writes catalog and collection documents only; item search is still served by the Element84 proxy. | `public.ecr.aws/eodh/element84-harvester` | `apps/resource-catalogue/base/proxies/element84/harvester.yaml` |
| [eodh-qa-data](https://github.com/EO-DataHub/eodh-qa-data) | QA documentation (Quality Processes Review) and radiometric uncertainty JSON files produced by NPL for EODH collections, exposed as `qa_documentation` and `qa_radiometric` STAC assets. |  |  |
| [eodh-stac-validator](https://github.com/EO-DataHub/eodh-stac-validator) | Validation service for ensuring STAC data conforms to specification standards before ingestion. | `public.ecr.aws/eodh/eodh-stac-validator` |  |
| [eodhp-init-catalogs-harvest](https://github.com/EO-DataHub/eodhp-init-catalogs-harvest) | Trigger harvesting of the top-level STAC Catalogs for loading into the Resource Catalogue. Currently this includes the `user-datasets` and `supported-datasets` top-level catalogs | `public.ecr.aws/n1b3o1k2//ukeodhpeodhp-init-catalog-ingest` | `apps/stac-fastapi-2/base/init-catalog-ingest.yaml` |
| [eodhp-stac-fastapi](https://github.com/EO-DataHub/eodhp-stac-fastapi) | Core FastAPI-based implementation of the STAC API for EODH | `public.ecr.aws/eodh/eodhp-stac-fastapi` | `apps/stac-fastapi-2/base/catalogue-search-service-ingester.yaml` |
| [eodhp-stac-fastapi-elasticsearch-opensearch](https://github.com/EO-DataHub/eodhp-stac-fastapi-elasticsearch-opensearch) | Elasticsearch and Opensearch backends for the stac-fastapi project. |  |  |
| [harvest-transformer](https://github.com/EO-DataHub/harvest-transformer) | A service for transforming harvested STAC metadata to make it suitable for the EODH catalogue. | `public.ecr.aws/eodh/harvest-transformer` | `apps/resource-catalogue/base/git-harvester/configuration-transformer.yaml`<br>`apps/resource-catalogue/base/transformer.yaml` |
| [opencosmos-harvester](https://github.com/EO-DataHub/opencosmos-harvester) | Harvests Open Cosmos archive imagery metadata, converts it to STAC, stores it in S3 and publishes change messages to Pulsar. Not yet referenced in ArgoCD. |  |  |
| [planet-harvester](https://github.com/EO-DataHub/planet-harvester) | Harvests collections from the Planet catalogue. Planet is large and contains many items - items need to be collected via the API | `public.ecr.aws/n1b3o1k2/planet-harvester` | `apps/resource-catalogue/base/planet/harvester.yaml` |
| [resource-catalog-support-utils](https://github.com/EO-DataHub/resource-catalog-support-utils) | A FastAPI web service that generates QLR (QGIS Layer) files from STAC items with pre-constructed XYZ tile service URL |  |  |
| [resource-catalogue-fastapi](https://github.com/EO-DataHub/resource-catalogue-fastapi) | An API to allow interaction with Stac-FastAPI within the EO Data Hub | `public.ecr.aws/eodh/resource-catalogue-fastapi` | `apps/resource-catalogue/base/workspaces/manage-catalogue-api.yaml` |
| [stac-fastapi-ingester](https://github.com/EO-DataHub/stac-fastapi-ingester) | An ingester to load STAC Catalogs, Collections and Items into STAC-Fastapi, read from S3 bucket, triggered by Pulsar messages. | `public.ecr.aws/n1b3o1k2/eodhp-stac-fastapi-ingester` | `apps/stac-fastapi-2/base/catalogue-search-service-ingester.yaml` |
| [stac-harvester](https://github.com/EO-DataHub/stac-harvester) | Harvests STAC catalogs from upstream sources. It collects STAC records from source catalogs, stores them in S3, and publishes Pulsar messages for added, changed, and deleted entries by comparing with stored hashes. | `public.ecr.aws/n1b3o1k2//ukeodhpstac-harvester` |  |
| [stac-harvester-configurations](https://github.com/EO-DataHub/stac-harvester-configurations) | Contains the configurations for harvesting STAC catalogues |  |  |
| [stac-harvester-ingester](https://github.com/EO-DataHub/stac-harvester-ingester) | Takes catalogue configuration data from a pulsar message and creates a STAC Harvester for each catalogue. | `public.ecr.aws/n1b3o1k2//ukeodhpstac-harvester-ingester` | `apps/resource-catalogue/base/git-harvester/configuration-ingester.yaml` |
| [stac-planet-api](https://github.com/EO-DataHub/stac-planet-api) | Converter from STAC request to Planet request then from Planet response to STAC response. | `public.ecr.aws/eodh/stac-planet-api` | `apps/resource-catalogue/base/planet/deployment.yaml` |
| [stac-pydantic](https://github.com/EO-DataHub/stac-pydantic) | Pydantic models for STAC Catalogs, Collections, Items, and the STAC API |  |  |
| [test-catalogue-data](https://github.com/EO-DataHub/test-catalogue-data) | A data repository for testing and experimentation. Assume that any and all branches may change. Used as the source data for the git harvester test job. |  | `apps/resource-catalogue/base/git-harvester/tests/files/harvester-test.py` |
| [workspace-catalog-generator](https://github.com/EO-DataHub/workspace-catalog-generator) | Contains code to generate required STAC catalogs for EODHP workspaces. | `public.ecr.aws/n1b3o1k2//ukeodhpeodhp-workspace-catalog-generator` | `apps/resource-catalogue/base/workspaces/workspaces-harvester.yaml` |
| [workspace-file-harvester](https://github.com/EO-DataHub/workspace-file-harvester) | Collects user-uploaded files from S3. STAC files are passed to the transformer and ingestor for harvesting and access policy files are used to update public access of files, folders and workflows. | `public.ecr.aws/n1b3o1k2/workspace-file-harvester` | `apps/resource-catalogue/base/workspaces/deployment.yaml` |

### Titiler

| Repository | Purpose | Docker image | ArgoCD |
|---|---|---|---|
| [titiler](https://github.com/EO-DataHub/titiler) | A modern dynamic tile server built on top of FastAPI and Rasterio/GDAL | `public.ecr.aws/eodh/titiler-core` | `apps/titiler/base/titiler/deployment.yaml` |
| [titiler-stacapi](https://github.com/EO-DataHub/titiler-stacapi) | Connect titiler to STAC APIs | `public.ecr.aws/eodh/titiler-stacapi` | `apps/titiler/base/stac-titiler/deployment.yaml` |

### Workflow Catalogue

| Repository | Purpose | Docker image | ArgoCD |
|---|---|---|---|
| [wf-catalogue-service](https://github.com/EO-DataHub/wf-catalogue-service) | Workflow and notebook catalogue API (OGC API – Records style). Lists catalogues and records, and registers or deletes workflow and notebook records. | `public.ecr.aws/eodh/wf-catalogue-service` | `apps/workflow-discovery/base/deployment.yaml` |
| [workflow-catalogue](https://github.com/EO-DataHub/workflow-catalogue) | Shared EODH workflow and notebook record definitions, with CI that validates them and CD that registers them in [wf-catalogue-service](https://github.com/EO-DataHub/wf-catalogue-service). |  |  |

### Workspaces

| Repository | Purpose | Docker image | ArgoCD |
|---|---|---|---|
| [eodhp-workspace-controller](https://github.com/EO-DataHub/eodhp-workspace-controller) | Provides reconcilers for a number of workspace configurations to ensure they remain aligned. This includes reconcilers for namespace, storage (EFS and S3), service, account & AWS IAM Policies |  |  |
| [eodhp-workspace-manager](https://github.com/EO-DataHub/eodhp-workspace-manager) | Bridges Kubernetes Workspace CRDs and Pulsar: it monitors workspace status changes and publishes them to Pulsar, and applies workspace configuration changes received from Pulsar by updating the Kubernetes Workspace resources. | `public.ecr.aws/eodh/eodhp-workspace-manager` | `apps/workspaces/base/manager/deployment.yaml` |
| [eodhp-workspace-services](https://github.com/EO-DataHub/eodhp-workspace-services) | An API gateway to the EO DataHub to manage workspace related API requests and process events for workspace management. | `public.ecr.aws/eodh/eodhp-workspace-services` | `apps/workspaces/base/api/` |

## Documentation

| Repository | Purpose |
|---|---|
| [eodh-training](https://github.com/EO-DataHub/eodh-training) | Training materials, tutorials, and examples for using the EODH platform and APIs. |
| [eodh-userdocs](https://github.com/EO-DataHub/eodh-userdocs) | Source for the user-facing EODH documentation site (MkDocs Material), published at `https://eodatahub.org.uk/docs/documentation`. |
| [eodhp-guide](https://github.com/EO-DataHub/eodhp-guide) | Guides to practical aspects of operating and developing the EO DataHub. |

## Examples

### General

| Repository | Purpose | Docker image |
|---|---|---|
| [eodh-qa-prototype](https://github.com/EO-DataHub/eodh-qa-prototype) | A prototype implementation of the EODH _QA Checker_ concept to illustrate the concept of the QA Checker and its interfaces. |  |
| [eodhp-local-dev-stack](https://github.com/EO-DataHub/eodhp-local-dev-stack) | Docker Compose stack that emulates S3 (RustFS) and Pulsar, with optional harvest-transformer and CWL runner containers, for local development of EODH backend services. |  |
| [template-python](https://github.com/EO-DataHub/template-python) | A template repository for new UKEODHP Python-base software components. | `public.ecr.aws/n1b3o1k2//ukeodhpCHANGEME` |

### Workflows

| Repository | Purpose | Docker image |
|---|---|---|
| [ceda-snuggs](https://github.com/EO-DataHub/ceda-snuggs) | Band math CWL workflow | `public.ecr.aws/eodh/ceda-snuggs` |
| [eodhp-ades-demonstration](https://github.com/EO-DataHub/eodhp-ades-demonstration) | Example Workflows, Notebooks and HTTP files to Demonstrate the EODH Workflow Runner |  |
| [example-workflows](https://github.com/EO-DataHub/example-workflows) | A number of workflows, defined as Common Workflow Language (CWL) scripts |  |
| [physrisk-workflow](https://github.com/EO-DataHub/physrisk-workflow) | A command-line interface tool for running Physrisk as a workflow, designed to assess physical climate risks for assets. |  |
| [public-workflows](https://github.com/EO-DataHub/public-workflows) | Publicly available workflows that can be executed by EODH platform users. |  |
| [tiff-to-cog-workflow](https://github.com/EO-DataHub/tiff-to-cog-workflow) | A CWL workflow for converting a remote TIFF file to a COG and generating a STAC catalog |  |
| [user-workflows](https://github.com/EO-DataHub/user-workflows) | Contains user specific processing workflows for the EO Data Hub. |  |
| [workflow-private-data-access](https://github.com/EO-DataHub/workflow-private-data-access) | An example Docker image and Common Workflow Language (CWL) Application Package that can be used with the EO DataHub Workflow Runner to demonstrate accessing private data within workflow steps |  |

## Frontend

| Repository | Purpose | Docker image | ArgoCD |
|---|---|---|---|
| [eodhp-rc-ui](https://github.com/EO-DataHub/eodhp-rc-ui) | Resource Catalogue User Interface |  |  |
| [eodhp-web-presence](https://github.com/EO-DataHub/eodhp-web-presence) | Public-facing UI using Django/Wagtail for content management, Webpack for frontend assets, and includes account management and public pages (home, documentation, accounts, workspaces) | `public.ecr.aws/eodh/eodhp-web-presence` | `apps/web-presence/base/jobs.yaml` |
| [eodhp-workspace-ui](https://github.com/EO-DataHub/eodhp-workspace-ui) | Frontend React/TypeScript application for managing EODH workspaces, including workspace selection/creation, member management, data loading/publishing, S3 storage, credentials, billing/invoices, catalog browsing, and linked accounts. |  |  |

## GitHub

| Repository | Purpose | Docker image |
|---|---|---|
| [.github](https://github.com/EO-DataHub/.github) | Storing shared or configuration-related files that affect how GitHub behaves for your organisation |  |
| [github-actions](https://github.com/EO-DataHub/github-actions) | Contains workflows for GitHub actions used by the EO-DataHub. See the [CI/CD reference](ci-cd/index.md) for the reusable workflows and shared Renovate configuration. |  |
| [link-checker](https://github.com/EO-DataHub/link-checker) | Standalone GitHub Action and CLI tool that checks websites for broken links using linkinator. | `public.ecr.aws/n1b3o1k2//ukeodhpCHANGEME` |

## Other

| Repository | Purpose |
|---|---|
| [eodh-community-notebook-competition](https://github.com/EO-DataHub/eodh-community-notebook-competition) | Rules, dates and submissions for the EODH Community Notebook Competition (October–December 2026). |
| [eodh-eocis-sprint](https://github.com/EO-DataHub/eodh-eocis-sprint) | Outputs from the EODH-EOCIS Sprints |

## Project management

| Repository | Purpose |
|---|---|
| [eodh-qa-pm](https://github.com/EO-DataHub/eodh-qa-pm) 🔒 | Quality assurance project management and testing coordination. |
| [eodhp-sparkgeo-pm](https://github.com/EO-DataHub/eodhp-sparkgeo-pm) 🔒 | SparkGeo related issue/project tracking |
| [feature-requests](https://github.com/EO-DataHub/feature-requests) 🔒 | Proposed new features for the EODH, and the main repository for the user journey project. |
| [platform-bugs](https://github.com/EO-DataHub/platform-bugs) 🔒 | This is a place for partners to report bugs and feature requests against the hub platform. |
| [project-delivery](https://github.com/EO-DataHub/project-delivery) 🔒 | Project delivery issues and tracking. |
| [project_management](https://github.com/EO-DataHub/project_management) 🔒 | Project management issues, and tracking for platform development. |
| [user-journeys](https://github.com/EO-DataHub/user-journeys) 🔒 | Tasks, issues and project tracking for the user journey work. |
| [user_engagement](https://github.com/EO-DataHub/user_engagement) 🔒 | Manage user engagement tasks |

## Tools

### Clarat

| Repository | Purpose |
|---|---|
| [climate-app-docs](https://github.com/EO-DataHub/climate-app-docs) | Documentation specific to climate applications and workflows within EODH. |
| [climate-app-lst-workflow](https://github.com/EO-DataHub/climate-app-lst-workflow) | Land Surface Temperature workflow implementation for climate applications. |
| [climate-app-ui](https://github.com/EO-DataHub/climate-app-ui) | User interface for climate-related applications and workflows within the EODH platform. |

### EOPro

| Repository | Purpose |
|---|---|
| [eodh-fe](https://github.com/EO-DataHub/eodh-fe) | Frontend for EOPro |
| [eodh-fe-infra](https://github.com/EO-DataHub/eodh-fe-infra) | Infrastructure for EOPro deployment |
| [eodh-workflows](https://github.com/EO-DataHub/eodh-workflows) | Workflows for Action Creator component on EOPro platform. |

### Integration

| Repository | Purpose |
|---|---|
| [arcgis-plugin](https://github.com/EO-DataHub/arcgis-plugin) | Early version of the EODH ArcGIS Pro add-in. Development continues in [eodh-arcgis](https://github.com/EO-DataHub/eodh-arcgis); no pushes since March 2026. |
| [dev-platform-eoap](https://github.com/EO-DataHub/dev-platform-eoap) | Kubernetes-based developer environment for Earth Observation Application Packages. Runs a browser-accessible code-server pod that can invoke Calrissian and submit Kubernetes Jobs to run CWL workflows. |
| [eoap-gen](https://github.com/EO-DataHub/eoap-gen) | A CLI tool for generating Earth Observation Application Packages including CWL workflows and Dockerfiles from user supplied python scripts. |
| [eodh-ac-api](https://github.com/EO-DataHub/eodh-ac-api) | API for interacting with Action Creator component and scheduling spatial computation in EO Data Hub. |
| [eodh-arcgis](https://github.com/EO-DataHub/eodh-arcgis) | ArcGIS Pro add-in to search, preview and load EODH datasets, purchase commercial imagery (Airbus, Planet) and load completed workspace assets. |
| [eodh-qgis](https://github.com/EO-DataHub/eodh-qgis) | QGIS plugin for integrating EODH platform data access within QGIS desktop applications. |
| [pyeodh](https://github.com/EO-DataHub/pyeodh) | A lightweight Python client for easy access to EODH APIs. |

## Archived

| Repository | Purpose |
|---|---|
| [documentation](https://github.com/EO-DataHub/documentation) 🔒 | Contains some of Telespazio's documentation about EODHP as a whole (vs an individual component) and working practices. |
| [eodhp-release-notes](https://github.com/EO-DataHub/eodhp-release-notes) 🔒 | Release notes for the work done by Telespazio UK on the EO DataHub platform. |
| [eodhp-sprint-reports](https://github.com/EO-DataHub/eodhp-sprint-reports) 🔒 | Contains reports on each Scrum sprint carried out by Telespazio UK for the EODH project. |
| [processes-catalog-api-proposal](https://github.com/EO-DataHub/processes-catalog-api-proposal) 🔒 | Proposed structures and documents for the EODH (Earth Observation Data Hub) process catalogue, which enables discovery and execution of Earth observation workflows and notebooks |
| [stac-browser](https://github.com/EO-DataHub/stac-browser) | STAC browser for static catalogs |
| [test-workflow-store](https://github.com/EO-DataHub/test-workflow-store) | Empty test repository for workflow storage. |
