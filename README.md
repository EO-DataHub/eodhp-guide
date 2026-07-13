# EO Data Hub Technical Guide

Technical documentation for the EO Data Hub platform, built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

## Getting started

Python 3.12 and [uv](https://docs.astral.sh/uv/) are required.

```bash
uv sync          # install dependencies
uv run mkdocs serve  # serve locally with live reload at http://127.0.0.1:8000
uv run mkdocs build  # build static site to site/
```

## Releasing a new version

1. Merge changes to `main`
2. Tag the release and push:
   ```bash
   git tag v1.x.x && git push origin v1.x.x
   ```
3. GitHub Actions builds the site and syncs it to S3 — verify it at `https://eodatahub.org.uk/static-apps/eodhp-guide/<version>/index.html`
4. Raise a PR to `eodhp-argocd-deployment` updating `EODHP_GUIDE_VERSION` in `apps/web-presence/base/kustomization.yaml`, then promote through Kargo

See [docs/how-to/deployments/deploy-eodhp-guide-to-kubernetes.md](docs/how-to/deployments/deploy-eodhp-guide-to-kubernetes.md) for the full setup guide.

## Content structure

All documentation lives under `docs/`.

| Folder | Contents |
|--------|----------|
| `docs/how-to/` | Step-by-step guides for operating and developing the platform |
| `docs/reference/` | Service pages, API overviews, repository inventories |
| `docs/explanation/` | Architecture, design decisions, background reading |

### Navigation

Navigation order is controlled by `.pages` files (via [mkdocs-awesome-pages-plugin](https://github.com/lukasgeiter/mkdocs-awesome-pages-plugin)). When adding or removing pages, update the `.pages` file in the same directory.

## Page frontmatter

Every page should include frontmatter with a title and review status:

```yaml
---
title: Page Title
doc_status: unreviewed
last_reviewed:
reviewed_by:
---
```

### `doc_status` values

| Value | Meaning |
|-------|---------|
| `unreviewed` | Not yet checked — use this as the default for new or migrated pages |
| `ok` | Reviewed and broadly correct |
| `needs-update` | Mostly valid but some content is stale or incomplete |
| `outdated` | Known to be wrong or significantly stale |
| `needs-expansion` | Accurate but too thin — missing examples, context, or detail |
| `needs-verification` | Looks plausible but needs confirming against code or product behaviour |
| `deprecated` | Describes something no longer in use, kept for historical reference |
| `remove-candidate` | Should probably be deleted, merged, or redirected |

For more detailed tracking, optional fields are available:

```yaml
---
title: Page Title
doc_status: needs-update
last_reviewed: 2026-05-19
reviewed_by: Alice Smith
review_notes: "CLI flags changed; examples need updating."
---
```

## Tags

Pages are tagged to make related content discoverable across sections. Tags appear on each page and are aggregated on the [Tags](docs/tags.md) page.

### Tag taxonomy

Two categories only — keeping it narrow prevents sprawl.

#### Service / technology tags

Use the canonical name (lowercase, hyphens). Tag the **primary** service a page is about; don't tag every service mentioned in passing.

| Tag | Covers |
|-----|--------|
| `argo-cd` | Argo CD (deployment, web UI, app syncing) |
| `kargo` | Kargo (promotion, freight, warehouses) |
| `keycloak` | Keycloak (auth, realms, clients, users) |
| `oauth2-proxy` | oauth2-proxy |
| `auth-agent` | auth-agent |
| `open-policy-agent` | OPA / OPAL |
| `grafana` | Grafana (dashboards, monitoring) |
| `prometheus` | Prometheus (metrics, scraping) |
| `victorialogs` | VictoriaLogs / Grafana (logging) |
| `linkerd` | Linkerd (service mesh, mTLS, certificates) |
| `jupyter` | JupyterHub / notebooks |
| `stac` | STAC FastAPI / resource catalogue |
| `rc-ui` | Resource Catalogue UI (eodhp-rc-ui) |
| `qgis` | QGIS layer integration |
| `titiler` | TiTiler (tile services, WMTS) |
| `pulsar` | Apache Pulsar (messaging) |
| `workspaces` | Workspaces service |
| `aws` | AWS-specific (EKS, EC2, IAM roles, SES, ECR) |
| `terraform` | Terraform (infrastructure provisioning) |
| `kubernetes` | Kubernetes (general cluster ops) |
| `oidc` | OIDC / OAuth2 protocol-level pages |

#### Topic tags

Cross-cutting themes that span multiple services or sections.

| Tag | Covers |
|-----|--------|
| `deployment` | Standing up or bootstrapping environments |
| `gitops` | Argo CD / Kargo branching & promotion model |
| `identity` | IAM concepts, user management, access policies |
| `observability` | Metrics, dashboards, logs |
| `data-catalogues` | Ingestion, harvesting, STAC catalogue management |
| `commercial-data` | Commercial data purchasing, orders, provider integrations |
| `notebooks` | Notebook images, workspace onboarding |
| `workflows` | Argo Workflows, workflow output processing |
| `security` | Certificates, mTLS rotation, trust anchors |
| `maintenance` | Upgrades, rotation, housekeeping |
| `analytics` | Google Analytics / GA4 |

### Rules

- **Max 4 tags per page.** Prefer fewer.
- **At least one service/tech tag** if the page is about a specific service.
- **Don't duplicate the section.** Don't tag every how-to page with `deployment` just because it's under `how-to/deployments/` — only tag if deployment is the actual subject.

## Diagrams

- **PlantUML** — use ` ```kroki-plantuml ` fenced code blocks, rendered via [kroki.io](https://kroki.io) at page-load time
- **Mermaid** — use ` ```mermaid ` fenced code blocks, rendered client-side via `pymdownx.superfences`
