# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

This is a Python/MkDocs documentation project managed with `uv`.

```bash
# Install dependencies
uv sync

# Serve locally with live reload
uv run mkdocs serve

# Build static site to site/
uv run mkdocs build
```

Python 3.12 is required (see `.python-version`).

## Architecture

### Content directory

`docs/` is the active content directory. `mkdocs.yaml` points `docs_dir` at `docs/`. The `site/` output directory is gitignored.

### Content organisation — Diátaxis framework

`docs/` is structured around the [Diátaxis](https://diataxis.fr/) framework:

| Folder | Put here |
|--------|----------|
| `docs/how-to/` | Task-oriented procedural guides ("restart X", "deploy Y") |
| `docs/reference/` | Neutral facts — catalogues, inventories, port lists, repo tables |
| `docs/explanation/` | Rationale, architecture, design trade-offs |
| `docs/architecture/` | System architecture documentation |

`how-to/` has topic subfolders: `deployments/`, `kubernetes-gitops/`, `observability-logging/`, `data-and-catalogues/`, `identity-access/`, `notebooks-and-workspaces/`, `analytics/`, `documentation/`.

When content spans quadrants, split into separate files linked by cross-reference rather than mixing modes in one page.

### Navigation — `.pages` files

Navigation order is controlled by `.pages` files (mkdocs-awesome-pages-plugin). Each directory that needs a custom nav order has one. When adding or removing files, update the `.pages` file in that directory to match. The format is:

```yaml
title: Section Title   # optional
nav:
  - Human Label: filename.md
  - subdirectory
```

### Page frontmatter

Every page should carry review-status frontmatter:

```yaml
---
title: Page Title
doc_status: unreviewed   # or ok, needs-update, outdated, needs-verification, etc.
last_reviewed:
reviewed_by:
review_notes:
---
```

### Diagrams

PlantUML diagrams are rendered via the `mkdocs-puml` plugin (remote render at plantuml.com). Mermaid diagrams are supported via `pymdownx.superfences`.

### Custom skill

A custom Claude Code skill (`/eodhp-docs2-placement`) is available to determine the correct `docs/` path for any piece of content. Use it when placing new pages.
