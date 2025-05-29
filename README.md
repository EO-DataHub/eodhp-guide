# EO DataHub Platform Guide

This guide will outline all processes for an administrator of the UK EO DataHub.

## Deployment Repositories

- https://github.com/EO-DataHub/eodhp-deploy-supporting-infrastructure - Initialises an AWS instance with any resources to be shared by deployed environments.
- https://github.com/EO-DataHub/eodhp-deploy-infrastucture - Manages infrastructure for an individual deployment environment, including initial deployment. It also bootstraps the Kubernetes cluster, including some required resources, drivers and operators.
- https://github.com/EO-DataHub/eodhp-argocd-deployment - Manages the deployment environment services and resources using GitOps.

## Deployment

See "Platform Deployment.md"
