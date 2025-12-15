# Kargo Deployment Workflow

## Accessing Kargo

- **URL**: `https://kargo.<domain>` (e.g., `https://kargo.eodatahub.org.uk` for production)
- **Authentication**: Keycloak OIDC via the `eodhp` realm
- **Authorization**: Users must be in the `admin` group for full access

## Key Concepts

### Configuration vs Image Versions

**Important distinction**:

| Change Type | How to Deploy |
|-------------|---------------|
| **Configuration changes** (manifests, settings, env vars) | PR to `main` branch in this repository, deployed via ArgoCD |
| **Image version updates** | Handled exclusively by Kargo |

Kargo only manages **which image versions** are deployed. All other application configuration remains on `main` branch of [eodhp-argocd-deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository.
## Warehouses

Each application has its own Warehouse that tracks:
- **Git**: The `main` branch of this repository (app paths and environment overlays)
- **Container images**
- **Helm charts**

### Automatic Freight Creation (Semver Tags)

When images have tags matching the semantic version pattern:
```
^v?[0-9]+\.[0-9]+\.[0-9]+
```
(e.g., `1.2.3` or `v1.2.3`), freight is created **automatically**.

This is enforced by the `freightCreationCriteria` expression in the Warehouse configuration.

### Dev Tags and Manual Freight Creation

In-house built images have `enableDevTags: true`, allowing non-semver tags such as:
- Commit hashes (e.g., `abc123f`)
- Feature branch identifiers
- Development builds

These tags do **NOT** trigger automatic freight creation due to the semver filter.

**To deploy a dev tag**:
1. Navigate to the application's Warehouse in the Kargo UI
2. Click **Refresh** to discover new image tags if warehouse has not been refreshed yet automatically
3. Click **Create Freight** and select the desired dev tag

### Configuration change of an application

Any change merged into `main` branch of [eodhp-argocd-deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment) creates a Freight and gets promoted to test automatically.

## Stage Pipeline

```
Warehouse → test (auto) → staging (manual) → prod (manual)
```

| Stage | Auto-Promotion | Description |
|-------|----------------|-------------|
| **test** | Yes | New freight automatically deploys when created |
| **staging** | No | Requires manual promotion from test |
| **prod** | No | Requires manual promotion from staging |

## Promotion Process

When a Freight is promoted to a stage, Kargo executes the following steps:

1. **git-clone**: Clone the repository
2. **kustomize-set-image**: Update container image digests
3. **yaml-update**: Update Helm chart versions (if applicable)
4. **kustomize-build**: Build final manifests
5. **git-commit**: Commit changes to `kargo/<app>/<env>` branch
6. **argocd-update**: Update ArgoCD Application to sync from the new revision


## Promoting Through Stages

**From the Kargo UI**:
1. Navigate to the application's project
2. Select the Stage you want to promote
3. Click **Promote from upstream** to advance it to the next stage


## Common Workflows

### Deploying a New Release (Semver)

1. Push a new semver-tagged image to ECR (e.g., `v1.2.3`)
    - Build the image from the repo
    - Push to ECR
    - This may happen automatically on pushes to some repo branches
2. Kargo automatically creates Freight
3. Freight auto-promotes to **test**
4. Verify in test environment
5. Manually promote to **staging**
    - Go to `https://kargo.<domain>`
    - Navigate to the project -> staging stage
    - Click 'Promote from upstream'
6. Verify in staging environment
7. Manually promote to **prod**
    - Navigate to the titiler project → prod stage
    - Click "Promote from upstream"

### Deploying a Dev Build

1. Push a dev-tagged image to ECR (e.g., commit hash)
2. Go to Kargo UI → Warehouse → **Refresh**
3. **Create Freight** with the dev tag
4. Freight auto-promotes to **test**
5. Test and iterate as needed

### Configuration Change (No Image Update)

1. Make changes in this repository
2. Create PR to `main` branch
3. Merge PR
4. Kargo automatically creates Freight
5. Freight auto-promotes to test
6. Verify in test environment
7. Manually promote to staging
8. Verify in staging environment
9. Manually promote to prod

## Configuration Reference

| File | Purpose |
|------|---------|
| `apps/kargo-eodhp-project/base/values.yaml` | Application list with image/chart subscriptions |
| `apps/kargo-eodhp-project/base/templates/warehouses.yaml` | Warehouse configuration |
| `apps/kargo-eodhp-project/base/templates/stages.yaml` | Stage definitions |
| `apps/kargo-eodhp-project/base/templates/project.yaml` | Auto-promotion policies |
| `apps/kargo-eodhp-project/base/templates/promotiontasks.yaml` | Promotion workflow |