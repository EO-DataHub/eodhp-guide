---
title: Kargo developer guide
doc_status: ok
tags:
  - kargo
  - argo-cd
  - gitops
last_reviewed: 2026-06-29
reviewed_by: recmanj
review_notes: "Split from docs/operations/kargo/developer-guide.md (TL;DR + sections 7–9)"
---

# Kargo developer guide

Steps for progressing freight, onboarding apps, and Kustomize overlays in the deployment repo.

**Design context:** **[Kargo and Argo CD — design on EO Data Hub](../../explanation/development-and-governance/kargo-argoc-integration.md)** · **[GitOps branching and promotion](../../explanation/development-and-governance/gitops-branching-and-kargo-promotion.md)**.

## TL;DR -- Quick Deployment Guide

### Config change

1. Commit to `main` (edit files under `apps/<app>/` or `eodhp/envs/*/vars.yaml`)
2. `<app>-config` warehouse detects the change and creates freight automatically
3. Auto-promotes to **test**
4. Manually promote **staging** then **prod** in the [Kargo UI](https://kargo.eodatahub.org.uk)

### New semver image

1. Push a semver-tagged image to ECR (e.g. `accounting-service:0.6.0`)
2. `<app>-images` warehouse detects the new tag (must match the `constraint`)
3. Auto-promotes to **test**
4. Manually promote **staging** then **prod** in the Kargo UI

### Dev / feature image

1. Push your image with any tag to ECR
2. In the Kargo UI, navigate to the `<app>-dev-images` warehouse
3. Refresh the warehouse, then click **Create Freight** -- you must **select image tags for ALL images** in the warehouse, not just the one you changed
4. Promote `<app>-config` freight that is currently promoted in test stage (skip this if the freight is already promoted in dev)
5. Promote the freight to the `<app>-dev` stage
6. The image deploys to the **test** environment

---

## Developer workflows

### Deploying a configuration change

1. Make your changes in `apps/<app>/` (manifests, values, patches) or in `eodhp/envs/*/vars.yaml`
2. Commit and push to `main`
3. The `<app>-config` warehouse detects the new commit and creates freight
4. The `<app>-test` stage auto-promotes the freight
5. Verify in the test environment
6. In the Kargo UI, promote the freight to `<app>-staging`, then `<app>-prod`

### Deploying a new semver image

1. Tag and push your image with a semver tag (e.g. `public.ecr.aws/eodh/accounting-service:0.6.0`)
2. The tag must match the `constraint` in `values.yaml` (e.g. `>=0.5.3`)
3. The `<app>-images` warehouse detects the new tag and creates freight
4. The `<app>-test` stage auto-promotes
5. Verify, then manually promote through staging and prod

### Deploying a dev/feature branch image

1. Push your image with any tag to ECR (e.g. `my-feature-branch`, `fix-123`, etc.)
2. Open the [Kargo UI](https://kargo.eodatahub.org.uk) and navigate to the `eodhp` project
3. Find the `<app>-dev-images` warehouse
4. Click **Refresh** to pick up the new tag
5. Click **Create Freight** -- **important:** you must select image tags for **every** image in the warehouse, not just the one you changed. The UI shows a dropdown per image subscription.
6. Promote the new freight to the `<app>-dev` stage
7. The `<app>-dev` stage deploys the image to the **test** environment

> **Why manual?** Dev images use `freightCreationPolicy: Manual` and `imageSelectionStrategy: NewestBuild` to avoid auto-detecting every push. This gives you explicit control over which image combination to deploy.

### Updating a static app version

Some services embed the version of a static frontend app as a plain ConfigMap value rather than a container image tag. These versions are **not** tracked by Kargo, so they are safe to edit directly and will not be overwritten on promotion. Examples in the `web-presence` app:

| Config key | App |
|---|---|
| `WORKSPACE_UI_VERSION` | [eodhp-workspace-ui](https://github.com/EO-DataHub/eodhp-workspace-ui) |
| `EODHP_GUIDE_VERSION` | [eodhp-guide](https://github.com/EO-DataHub/eodhp-guide) |
| `RESOURCE_CATALOGUE_VERSION` | [eodhp-resource-catalogue-ui](https://github.com/EO-DataHub/eodhp-resource-catalogue-ui) |

To update one of these:

1. Edit the value in `apps/web-presence/base/kustomization.yaml`
2. Also update the matching JSON patch in `apps/web-presence/envs/<env>/kustomization.yaml` for **every** environment that overrides the base value (currently test and staging both have their own patch — if you only update the base, the env-level patch takes precedence and the base change has no effect for those environments)
3. Commit to `main`
4. The `web-presence-config` warehouse detects the change and auto-promotes to test
5. Verify, then promote staging and prod via the [Kargo UI](https://kargo.eodatahub.org.uk)

The static app's files must already be present in S3 at the target version before promoting beyond test. See [Deploying a new workspace-ui version](../../reference/services/web-presence.md#deploying-a-new-workspace-ui-version) for the full two-step process.

### Rolling back (promote forward)

Kargo has no dedicated rollback feature. Instead, you "roll back" by **re-promoting an earlier freight** to the target stage. From Kargo's perspective this is just another promotion.

1. Open the [Kargo UI](https://kargo.eodatahub.org.uk) and navigate to the `eodhp` project
2. Click on the stage you want to roll back (e.g. `<app>-staging`)
3. In the freight timeline, find the older freight that was previously running
4. Click the freight and choose **Promote** to re-promote it to the stage

#### Manual steps

Some rollbacks may require additional manual steps beyond re-promoting freight — for example, reverting database migrations or restoring external state. Check your application's specific requirements before rolling back.

> **Further reading:** See the [Kargo documentation](https://docs.kargo.io) for more details on freight management and promotions.

---

## Adding a new application

### Step 1: Create the app directory structure

```
apps/<app-name>/
  base/
    kustomization.yaml    # namespace, resources, generators, patches
    namespace.yaml        # Namespace resource
    deployment.yaml       # (or other manifests)
    ...
  envs/
    test/
      kustomization.yaml  # resources: [../../base]
    staging/
      kustomization.yaml  # resources: [../../base]
    prod/
      kustomization.yaml  # resources: [../../base]
```

Apps have overlays only for `test`, `staging`, and `prod`. There is no `dev` overlay — the `<app>-dev` Kargo stage deploys to the **test** environment (`targetEnv: test`), so it reuses the test overlay.

See [Environment overlays (Kustomize)](#environment-overlays-kustomize) for details on the kustomization files.

### Step 2: Add an entry to `apps/kargo-eodhp-project/base/values.yaml`

Add your application to the `applications` list:

```yaml
applications:
  # ...existing apps...

  - name: my-app
    path: apps/my-app
```

### All available options

```yaml
- name: my-app                    # (required) Application name; used in warehouse/stage names
  path: apps/my-app               # (required) Path to app directory in this repo
  stages:                         # (optional) Limit to specific environments
    - test                         #   Defaults to all environments if omitted
    - staging                      #   Order determines promotion chain
    - prod

  images:                          # (optional) Container images to track
    - repoURL: public.ecr.aws/eodh/my-image   # (required) Full image repo URL (no tag)
      constraint: ">=1.0.0"        # (optional) Semver constraint for the images warehouse
      semverConstraint: ">=1.0.0"  # (optional) Alias for constraint
      enableDevTags: true          # (optional) Create a dev-images warehouse subscription
                                   #   and a <app>-dev stage; default: false
      imageSelectionStrategy: Semver  # (optional) see https://docs.kargo.io/user-guide/how-to-guides/working-with-warehouses/#image-selection-strategies
      allowTags: "^v\\d+\\.\\d+$"  # (optional) Regex to filter discovered tags
      ignoreTags:                  # (optional) List of tags to exclude
        - latest
        - dev
      strictSemvers: true          # (optional) Only match strict semver tags
      discoveryLimit: 10           # (optional) Max tags to discover; default: 5
      customPath:                  # (optional) For CRDs or non-standard image fields
        file: base/values.yaml     #   File path (relative to app path) containing the image ref
        key: hub.image.tag         #   YAML key path to update
        valueType: tag             #   "tag" = write just the tag
                                   #   "full" (default) = write <repoURL>:<tag>

  charts:                          # (optional) Helm charts to track
    - name: my-chart               # (optional) Chart name; required for non-OCI repos
      repoURL: https://charts.example.com  # (required) Helm repo or OCI registry URL
      constraint: "^1.0.0"        # (optional) Semver constraint
      allowTags: "^v\\d+"         # (optional) Tag filter regex
      ignoreTags:                 # (optional) Tags to exclude
        - beta
      strictSemvers: true         # (optional) Strict semver matching
      discoveryLimit: 5           # (optional) Max versions to discover
      generatorFile: base/my-chart.yaml           # (option A) Single HelmChartInflationGenerator file
      generatorFiles:                              # (option B) Multiple generator files
        - base/my-chart-platform.yaml
        - base/my-chart-workspaces.yaml

  releases:                        # (optional) GitHub release tags to track
    - name: my-release             # (required) Release name identifier
      repoURL: https://github.com/org/repo.git   # (required) Git repo URL
      constraint: "^1.9.0"        # (optional) Semver constraint on release tags
      updates:                     # (required) How to update files when a new release is found
        - file: base/kustomization.yaml    # File to update (relative to app path)
          key: resources.1                  # YAML key path to update
          urlPrefix: "https://raw.githubusercontent.com/org/repo/"   # URL prefix before tag
          urlSuffix: "/manifests/install.yaml"                       # URL suffix after tag
```

### Examples from existing apps

**Simple image-only app** (e.g. `auth-agent`):
```yaml
- name: auth-agent
  path: apps/auth-agent
  images:
    - repoURL: public.ecr.aws/eodh/eodhp-auth-agent
      constraint: ">=0.5.2"
      enableDevTags: true
```

**Chart-only app** (e.g. `nginx`):
```yaml
- name: nginx
  path: apps/nginx
  charts:
    - name: nginx-ingress
      repoURL: https://helm.nginx.com/stable
      constraint: ^2.1.0
      generatorFile: base/nginx-ingress-chart.yaml
```

**Images with custom paths** (e.g. `jupyter`):
```yaml
- name: jupyter
  path: apps/jupyter
  images:
    - repoURL: public.ecr.aws/eodh/eodh-jupyter-hub
      constraint: ^4.2.0-0.2.0
      enableDevTags: true
      customPath:
        file: base/values.yaml
        key: hub.image.tag
        valueType: tag
    - repoURL: public.ecr.aws/eodh/eodh-default-notebook
      enableDevTags: true
      customPath:
        file: base/values.yaml
        key: singleuser.image.tag
        valueType: tag
  charts:
    - name: jupyterhub
      repoURL: https://hub.jupyter.org/helm-chart/
      constraint: ^4.1.0
      generatorFile: base/jupyterhub-chart.yaml
```

**GitHub releases** (e.g. `argo-events`):
```yaml
- name: argo-events
  path: apps/argo-events
  releases:
    - name: argo-events
      repoURL: https://github.com/argoproj/argo-events.git
      constraint: "^1.9.0"
      updates:
        - file: base/kustomization.yaml
          key: resources.1
          urlPrefix: "https://raw.githubusercontent.com/argoproj/argo-events/"
          urlSuffix: "/manifests/install.yaml"
```

**Prod-only app** (e.g. `kargo-eodhp-project`):
```yaml
- name: kargo-eodhp-project
  path: apps/kargo-eodhp-project
  stages: [prod]
```

**Environment-specific chart paths** (e.g. `stac-fastapi-2`):
```yaml
- name: stac-fastapi-2
  path: apps/stac-fastapi-2
  charts:
    - name: elasticsearch
      repoURL: https://helm.elastic.co
      constraint: ^8.5.1
      generatorFile: envs/{{env}}/elasticsearch-chart.yaml   # {{env}} is replaced per environment
```

---

## Environment overlays (Kustomize)

Each application uses [Kustomize overlays](https://kubectl.docs.kubernetes.io/references/kustomize/glossary/#overlay) to configure environments separately. The shared configuration lives in `apps/<app>/base/`, and each environment extends it with an overlay in `apps/<app>/envs/<env>/`.

The base `kustomization.yaml` declares the namespace, resources, generators, and patches that are common to all environments. Each environment overlay pulls in the base and can add environment-specific customisations -- image tag overrides, JSON patches, extra resources, or different Helm chart values.

A minimal overlay just inherits the base with no changes:

```yaml
# apps/<app>/envs/test/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base
```

To customise a specific environment, add `images`, `patches`, `generators`, or additional `resources` to that overlay's `kustomization.yaml`. This keeps environment differences isolated and explicit -- you can see exactly what differs between test, staging, and prod by looking at their respective overlay files.

Values that vary by environment (domains, AWS account IDs, database URLs) are handled separately via gomplate variables (`${[.vars.<path>]}`), which are substituted at deploy time from `eodhp/envs/<env>/vars.yaml`.

### Helm charts via HelmChartInflationGenerator

Applications that use Helm charts render them through Kustomize's `HelmChartInflationGenerator` rather than having Argo CD manage them as a separate Helm source. This lets us treat chart-rendered manifests the same as any other Kustomize resource -- they can be patched, combined with plain YAML resources, and processed through the same overlay pipeline. Chart definitions are listed under `generators` in the `kustomization.yaml` and point to a `*-chart.yaml` file:

```yaml
# apps/<app>/base/my-chart.yaml
apiVersion: builtin
kind: HelmChartInflationGenerator
metadata:
  name: my-chart
name: my-chart
repo: https://charts.example.com
version: 1.2.3                    # updated by Kargo promotions
releaseName: my-chart
namespace: my-app
valuesFile: values.yaml
```

For environment-specific chart configuration, place both the chart definition and values file in `envs/<env>/` instead of `base/`.
