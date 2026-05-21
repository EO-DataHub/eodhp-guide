# EO Data Hub Technical Guide

Technical documentation for the EO Data Hub platform, built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

## Getting started

Python 3.12 and [uv](https://docs.astral.sh/uv/) are required.

```bash
uv sync          # install dependencies
uv run mkdocs serve  # serve locally with live reload at http://127.0.0.1:8000
uv run mkdocs build  # build static site to site/
```

## Content structure

All documentation lives under `docs2/`. The `docs/` directory is legacy source kept for reference — do not edit it.

| Folder | Contents |
|--------|----------|
| `docs2/how-to/` | Step-by-step guides for operating and developing the platform |
| `docs2/reference/` | Service pages, API overviews, repository inventories |
| `docs2/explanation/` | Architecture, design decisions, background reading |

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

## Diagrams

- **PlantUML** — rendered via `mkdocs-puml` (remote render at plantuml.com)
- **Mermaid** — supported via `pymdownx.superfences` fenced code blocks
