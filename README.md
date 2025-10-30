# EO DataHub Platform Guide

This contains guides to practical aspects of operating and developing the EO DataHub.

The Architecture and Design Document and to a lesser extent the documentation repo provide more general architecture and design information.

## Deployment Repositories

- <https://github.com/EO-DataHub/eodhp-deploy-supporting-infrastructure> - Initialises an AWS instance with any resources to be shared by deployed environments.
- <https://github.com/EO-DataHub/eodhp-deploy-infrastucture> - Manages infrastructure for an individual deployment environment, including initial deployment. It also bootstraps the Kubernetes cluster, including some required resources, drivers and operators.
- <https://github.com/EO-DataHub/eodhp-argocd-deployment> - Manages the deployment environment services and resources using GitOps.

## Deployment

See "Platform Deployment.md"

## Repository Catalog

See [repositories.md](repositories.md) for a complete catalog of all EODH repositories organized by category.
