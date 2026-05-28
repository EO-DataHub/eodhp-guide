---
title: Deploy eodhp-guide to the platform
doc_status: unreviewed
last_reviewed:
reviewed_by:
review_notes:
tags:
  - deployment
  - s3
  - cloudfront
  - mkdocs
---

# Deploy eodhp-guide to the platform

This guide covers how to publish this MkDocs site as a versioned static site on the EODH platform, accessible at:

```
https://eodatahub.org.uk/static-apps/eodhp-guide/<version>/index.html
```

**How serving works:** CloudFront has an ordered cache behaviour that routes `/static-apps/*` directly to the `static-web-artefacts-eodh` S3 bucket (defined in `eodhp-deploy-infrastucture/terraform/cloudfront.tf`). No Kubernetes pods are involved in serving. The `web-presence` kustomization in `eodhp-argocd-deployment` tracks the live version so the platform knows which version to link to.

The deployment pattern follows `eodhp-workspace-ui`:

1. Tag the repo to trigger a GitHub Actions workflow
2. The workflow builds the static site and syncs it to S3 at a versioned path
3. Raise a PR to `eodhp-argocd-deployment` to update the live version pointer in `web-presence`

## Prerequisites

- AWS OIDC role `GitHubAccessEODH` already exists (created in `eodhp-deploy-supporting-infrastructure`) and grants any `EO-DataHub/*` GitHub repo write access to `static-web-artefacts-eodh`
- GitHub repository variables must be set on `EO-DataHub/eodhp-guide` (see Step 2)

## Step 1 — Update mkdocs.yaml for S3 hosting

CloudFront serves S3 objects by exact key. Without S3 static website hosting, a request for `/static-apps/eodhp-guide/1.0.0/how-to/` will not automatically resolve to `/index.html`. Adding `use_directory_urls: false` tells MkDocs to generate flat `.html` files (`how-to/deployments.html`) instead of directory-style paths (`how-to/deployments/index.html`), which CloudFront can serve directly.

Add this line to `mkdocs.yaml`:

```yaml
use_directory_urls: false
```

!!! note
    This changes local `mkdocs serve` URLs from `/how-to/deployments/` to `/how-to/deployments.html`. Navigation links within the site still work correctly — only the URL style changes.

## Step 2 — Set GitHub repository variables

In the `EO-DataHub/eodhp-guide` repository settings, add the following Actions variables:

| Variable | Value |
|---|---|
| `S3_PATH` | `s3://static-web-artefacts-eodh/static-apps/eodhp-guide` |
| `AWS_REGION` | `eu-west-2` |
| `AWS_ROLE_ARN` | ARN of the `GitHubAccessEODH` role (look up in AWS IAM console) |

## Step 3 — Add the GitHub Actions publish workflow

Create `.github/workflows/publish.yml` in the repo root:

```yaml
name: Publish to S3 bucket

on:
  workflow_call:
  push:
    tags:
      - v\d.*

permissions:
  contents: read

jobs:
  prechecks:
    name: Prechecks
    runs-on: ubuntu-latest
    steps:
      - run: |
          RETURN_CODE=0
          if [ -z "${{ vars.S3_PATH }}" ]; then
            echo "Github vars.S3_PATH required for publish"
            RETURN_CODE=1
          fi
          if [ -z "${{ vars.AWS_REGION }}" ]; then
            echo "Github vars.AWS_REGION required for publish"
            RETURN_CODE=1
          fi
          if [ -z "${{ vars.AWS_ROLE_ARN }}" ]; then
            echo "Github vars.AWS_ROLE_ARN required for publish"
            RETURN_CODE=1
          fi
          exit $RETURN_CODE

  tag:
    uses: EO-DataHub/github-actions/.github/workflows/get-version-tag.yaml@main
    with:
      github-ref: ${{ github.ref_name }}

  build:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: actions/checkout@v4

      - name: Install uv
        uses: astral-sh/setup-uv@v5

      - name: Install dependencies
        run: uv sync --frozen --no-dev

      - name: Build static site
        run: uv run mkdocs build --strict

      - name: Upload site to GitHub artifact
        uses: actions/upload-artifact@v4
        with:
          name: eodhp-guide
          path: site/
          retention-days: 1

  publish:
    name: Publish to S3 bucket
    needs: [prechecks, tag, build]
    uses: EO-DataHub/github-actions/.github/workflows/s3-publish.yaml@main
    with:
      app_artifact: eodhp-guide
      s3_path: ${{ vars.S3_PATH }}/${{ needs.tag.outputs.version }}
      aws_region: ${{ vars.AWS_REGION }}
      aws_role_arn: ${{ vars.AWS_ROLE_ARN }}
    permissions:
      id-token: write
      contents: read
```

## Step 4 — Tag and publish a release

```bash
git tag v1.0.0
git push origin v1.0.0
```

The workflow runs, builds the site, and syncs it to:

```
s3://static-web-artefacts-eodh/static-apps/eodhp-guide/1.0.0/
```

The site is then accessible at:

```
https://eodatahub.org.uk/static-apps/eodhp-guide/1.0.0/index.html
```

Verify it is accessible before proceeding to Step 5.

## Step 5 — Update the live version pointer in eodhp-argocd-deployment

Raise a PR against `eodhp-argocd-deployment` to update the version tracked by `web-presence`. Edit `apps/web-presence/base/kustomization.yaml` and add (or update) these two lines in the `configMapGenerator` literals:

```yaml
- EODHP_GUIDE_URL=https://${[.vars.platform.domain]}/static-apps/eodhp-guide
- EODHP_GUIDE_VERSION=1.0.0
```

This follows the same pattern as `WORKSPACE_UI_URL`/`WORKSPACE_UI_VERSION` already present in that file.

!!! note "Django web-presence code"
    The argocd kustomization update records the live version in the GitOps config. The versioned S3 URL is accessible immediately. For the friendly `/eodhp-guide/` URL to work, a separate code change to `eodhp-web-presence` is needed — see Step 6.

Merge and promote the PR through Kargo as normal for a `web-presence` change.

## Step 6 — Add the guide link to eodhp-web-presence

The versioned S3 URL works immediately after Step 4. The friendly URL `https://eodatahub.org.uk/eodhp-guide/` requires changes to both `eodhp-web-presence` (a redirect view) and `eodhp-argocd-deployment` (an auth-bypass ingress).

### eodhp-web-presence changes

1. **`settings.py`** — add an `EODHP_GUIDE` dict after the existing `WORKSPACE_UI` block, using the same `env()` pattern:

    ```python
    EODHP_GUIDE = {
        "version": env("EODHP_GUIDE_VERSION", default="v1.0.0"),
        "url": env("EODHP_GUIDE_URL", default=None),
    }
    ```

2. **`views.py`** — add a redirect view (the guide is a static MkDocs site, not an SPA, so a redirect to the versioned S3 path is correct). CloudFront does not serve directory indexes, so `index.html` must be explicit:

    ```python
    def eodhp_guide_page_view(request: HttpRequest) -> HttpResponse:
        return redirect(
            "{url}/{version}/index.html".format(
                url=settings.EODHP_GUIDE["url"],
                version=settings.EODHP_GUIDE["version"],
            )
        )
    ```

3. **`urls.py`** — import `eodhp_guide_page_view` and register the URL pattern:

    ```python
    path("eodhp-guide/", eodhp_guide_page_view),
    ```

4. Raise a PR to `eodhp-web-presence`, tag a release, and raise a separate PR to `eodhp-argocd-deployment` updating the `web-presence` image tag. Promote through Kargo as normal.

### eodhp-argocd-deployment changes

The platform nginx master ingress applies `auth_request` to every location by default. The guide is public documentation, so `/eodhp-guide/` must bypass authentication. Add a second ingress object to `apps/web-presence/base/ingress.yaml`:

```yaml
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.org/mergeable-ingress-type: minion
    nginx.org/location-snippets: |
      auth_request off;

      include blocks/header_guards.conf;
  name: web-presence-eodhp-guide
  namespace: web
spec:
  ingressClassName: nginx
  rules:
    - host: ${[.vars.platform.domain]}
      http:
        paths:
          - backend:
              service:
                name: web-presence
                port:
                  number: 8000
            path: /eodhp-guide/
            pathType: Prefix
```

Without this, unauthenticated users are redirected to the login page before the Django redirect fires.

## Releasing a new version

1. Make changes and merge to `main`
2. Tag the new release: `git tag v1.1.0 && git push origin v1.1.0`
3. Verify `https://eodatahub.org.uk/static-apps/eodhp-guide/1.1.0/index.html` is live
4. Raise a PR to `eodhp-argocd-deployment` updating `EODHP_GUIDE_VERSION` to `1.1.0`

Old versions remain accessible at their versioned URLs until manually removed from S3.

## Expected URLs

| Environment | URL | Notes |
|---|---|---|
| Production (versioned) | `https://eodatahub.org.uk/static-apps/eodhp-guide/1.0.0/index.html` | Available after Step 4 |
| Staging (versioned) | `https://staging.eodatahub.org.uk/static-apps/eodhp-guide/1.0.0/index.html` | Available after Step 4 |
| Test (versioned) | `https://test.eodatahub.org.uk/static-apps/eodhp-guide/1.0.0/index.html` | Available after Step 4 |
| Production (friendly) | `https://eodatahub.org.uk/eodhp-guide/` | Requires Step 6 |

!!! note "Staging and test environments"
    The `static-web-artefacts-eodh` bucket is shared across environments. Each environment's CloudFront distribution has a separate `/static-apps/*` origin pointing to the same bucket, so a version published once is available in all environments at its versioned path.
