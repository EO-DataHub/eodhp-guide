---
title: EO Data Hub Technical Guide (Diátaxis)
doc_status: unreviewed
last_reviewed:
reviewed_by:
review_notes: Portal for docs2 scaffold; migrate content from docs/ incrementally.
---
# EO Data Hub Technical Guide

This tree follows [Diátaxis](https://diataxis.fr/): tutorials, how-to guides, reference, and explanation—as **modes of engagement**, not separate audiences.

| Section | Purpose |
|---------|---------|
| [Tutorials](tutorials/index.md) | Guided learning paths; keep short and reproducible |
| [How-to guides](how-to/index.md) | Task-focused procedures and runbooks |
| [Reference](reference/index.md) | Factual lookup: services, repos, APIs, data |
| [Explanation](explanation/index.md) | Design, rationale, and system understanding |
| [Contributing](contributing/index.md) | Documentation and programme workflows (outside the four quadrants) |
| [Governance](governance/index.md) | Organisation–level policies (GitHub org, membership) |

Content is migrating from [`docs/`](../docs/README.md): move Markdown files manually and fix internal links. When switching the published site over, set MkDocs **`docs_dir` to `docs2`** (or equivalent) and either copy or symlink `stylesheets/` and `assets/` from `docs/` if they remain there, since theme paths (`extra_css`, `logo`) are resolved relative to the documentation root.
