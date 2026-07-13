---
title: Minimal test deployment
doc_status: unreviewed
tags:
  - deployment
  - aws
  - kubernetes
  - argocd
  - kargo
  - keycloak
last_reviewed:
reviewed_by:
review_notes:
---

# Minimal test deployment

This page gives an orientation to deploying a test EODHP system from scratch. It is an overview of the phases involved, not a step-by-step guide — detailed instructions are in the linked how-to pages. Expect to spend the better part of a day on a first deployment.

!!! warning "This is a substantial undertaking"
    Standing up EODHP requires co-ordinating several repositories, AWS infrastructure, and manual configuration steps. There are race conditions during bootstrapping and environment-specific issues you will likely need to work through. This overview gives you the map; the linked guides have the detail.

## Prerequisites

Before starting, confirm you have:

- **AWS account** with permissions to create EKS clusters, VPCs, Route53 records, IAM roles, and S3 buckets
- **Domain and DNS control** — the platform deploys to a known hostname; you need to manage DNS for it
- **Tooling installed**: Terraform CLI, kubectl, AWS CLI, aws-iam-authenticator, Kustomize, Helm, Gomplate
- **DockerHub credentials** for ECR pull-through cache configuration

---

## Phase 1 — Infrastructure

**Repos:** `eodhp-deploy-supporting-infrastructure`, `eodhp-deploy-infrastucture`

Two Terraform deployments create the AWS foundation:

1. **Shared infrastructure** (`eodhp-deploy-supporting-infrastructure`) — deployed once per AWS account. Creates the VPC, internet gateways, NATs, RDS database server, and shared S3 buckets.
2. **Cluster infrastructure** (`eodhp-deploy-infrastucture`) — deployed once per environment. Creates the EKS cluster, CloudFront distributions, Route53 records, node groups, IAM roles, and ECR pull-through cache.

**Done when:** `kubectl config get-contexts` shows the new cluster and `kubectl get nodes` returns healthy nodes.

**Detailed steps:** [Platform deployment — Terraform section](platform-deployment.md#terraform)

---

## Phase 2 — Platform bootstrap (ArgoCD)

**Repo:** `eodhp-argocd-deployment`

ArgoCD manages all EODHP service deployments via GitOps. On first deployment, ArgoCD itself must be bootstrapped before Kargo-managed branches exist.

The bootstrap deploys a core set of platform services: ArgoCD, cluster autoscaler, cert-manager, nginx ingress, external-secrets, Redis, Pulsar, and the database operator. These are the dependencies that everything else relies on.

!!! note
    Race conditions during initial sync are normal. If apps fail to sync, force-sync them individually from the ArgoCD UI. See [Bootstrap dependencies](bootstrap-dependencies.md) for sync-wave ordering and known constraints.

**Done when:** All core applications in the ArgoCD UI show `Synced` and `Healthy`. The ArgoCD UI is reachable at your configured hostname.

**Detailed steps:** [Platform deployment — ArgoCD section](platform-deployment.md#argocd)

---

## Phase 3 — Identity (Keycloak + Kargo)

**Repos:** `eodhp-argocd-deployment`, `eodhp-opa-config`

Before Kargo can promote applications, Keycloak must be configured. This phase involves manual steps in the Keycloak admin UI to:

- Create an admin user in the master realm
- Configure an ArgoCD OIDC client (so ArgoCD UI authenticates via Keycloak)
- Configure a Kargo OIDC client in the `eodhp` realm
- Copy generated OIDC client secrets into the AWS `eodhp` secret store
- Ensure an OPA config branch exists for the cluster

Once complete, Kargo can be used to promote all applications from `main` to their `kargo/<app>/<env>` branches, switching ArgoCD from bootstrap mode to normal GitOps operation.

For background on how Kargo and ArgoCD interact in this platform, see [Kargo and Argo CD — design on EO Data Hub](../../explanation/development-and-governance/kargo-argoc-integration.md).

**Done when:** You can log in to the Kargo UI at its configured hostname using Keycloak credentials, and all applications have been promoted through Kargo.

**Detailed steps:** [Platform deployment — Keycloak and Kargo sections](platform-deployment.md#keycloak-configuration)

---

## Phase 4 — Services (catalogue + compute)

**Repos:** managed via `eodhp-argocd-deployment` overlays

With the platform running and Kargo managing promotions, the EODHP services deploy automatically as ArgoCD syncs. The core services for a minimal test system are:

- **Workspace management** — creates and manages the S3/NFS storage and Kubernetes resources for each user workspace. Most other services depend on this being healthy.
- **Resource catalogue** — STAC API backed by catalogue ingesters and transformers. Provides the core data discovery endpoint.
- **ENS (Argo Events + Workflows)** — event and scheduling system that triggers catalogue harvesters on a schedule. Without this the catalogue will not be populated with data.
- **Data Access Services** — TiTiler (tile server for visualising raster data) and HTTPS Download (Lambda + nginx for direct file access from workspace storage). Without these the catalogue is queryable but data cannot be accessed or visualised.
- **ADES + Workflow API** — the CWL-based workflow execution engine (from EOEPCA) and the EODHP API wrapper around it. Required to run user-defined processing workflows against catalogue data.
- **JupyterHub** — notebook environment for users, deployed into the workspace infrastructure.

**Done when:**

- The STAC API responds at its configured endpoint and returns catalogue entries (not just an empty root — harvesters need time to run)
- JupyterHub is reachable and launches a JupyterLab session in a user workspace
- A test workflow can be submitted via the Workflow API and executes via ADES

---

## Phase 5 — Observability

**Repos:** managed via `eodhp-argocd-deployment` overlays

A test system without observability makes diagnosing problems significantly harder. The platform ships with:

- **Prometheus + Grafana** — metrics collection and dashboards for cluster and service health
- **VictoriaLogs** — log aggregation for all platform services

These are optional for a test system but strongly recommended — most deployment issues manifest first in logs or metrics rather than in user-visible errors.

**Done when:** Grafana is reachable at its configured hostname and service dashboards are populating with data.

---

## Phase 6 — Frontends

Three user-facing web applications complete the platform. Note that `eodhp-rc-ui` has a different deployment path to the others — it is deployed to S3/CloudFront via AWS CDK rather than via ArgoCD.

**`eodhp-rc-ui` (Resource Catalogue UI)** — the primary interface for discovering, browsing, and visualising geospatial data. A React SPA deployed to S3 and served via CloudFront. Depends on the STAC API, TiTiler, Keycloak, and the Workspace API. Deployed using AWS CDK from the `iac/` directory of the repository.

**`workspace-ui`** — client-side application for managing workspaces, viewing usage data, and initiating workflows. Deployed via ArgoCD.

**`eodhp-web-presence` (Wagtail CMS)** — the public-facing web presence and documentation site. Deployed via ArgoCD.

For a test system validating API and notebook access, these are optional — but the RC UI is the most useful of the three to bring up early, as it provides a graphical view of the catalogue.

**Done when:** The RC UI loads at its configured hostname and can browse STAC collections. The Workspace UI can connect to the workspace management API.

**Reference:** [Resource Catalogue UI](../../reference/services/resource-catalogue-ui.md)

---

## Next steps

Once the platform is running, see:

- [Kargo developer guide](../kubernetes-gitops/kargo-developer-guide.md) — how to promote service updates through environments
- [Platform deployment](platform-deployment.md) — full step-by-step reference for all of the above
