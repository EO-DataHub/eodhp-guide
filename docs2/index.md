---
title: EO Data Hub Technical Guide (Diátaxis)
doc_status: unreviewed
last_reviewed: 2026-05-20
reviewed_by: geodowd
review_notes: "Portal for docs2 scaffold; migrate content from docs/ incrementally."
---
# EO Data Hub Technical Guide

These guides focus on practical aspects of **operating** and **developing** the EO Data Hub—runbooks for delivery as well as design context for builders.

For **system design and architecture**, start under [Explanation → Architecture](explanation/architecture/index.md). Additional historical design narrative may still live adjacent documentation repositories referenced from those sections.

**Deployment infrastructure** is described repository-by-repository in [Deployment repositories](reference/deployment-repositories.md); step-by-rollout procedures move into [Deployments how-to](how-to/deployments/index.md).

The **catalogue of EODH-related GitHub repositories** is in [Repositories](reference/repositories.md).

This tree follows [Diátaxis](https://diátaxis.fr/): tutorials, how-to guides, reference, and explanation—as **modes of engagement**, not separate audiences.

| Section | Purpose |
|---------|---------|
| [Tutorials](tutorials/index.md) | Guided learning paths; keep short and reproducible |
| [How-to guides](how-to/index.md) | Task-focused procedures and runbooks |
| [Reference](reference/index.md) | Factual lookup: services, repos, APIs, data |
| [Explanation](explanation/index.md) | Design, rationale, and system understanding |
| [Contributing](contributing/index.md) | Documentation and programme workflows (outside the four quadrants) |
| [Governance](governance/index.md) | Organisation–level policies (GitHub org, membership) |

Content is migrating from [`docs/`](../docs/README.md): move Markdown files manually and fix internal links. When switching the published site over, set MkDocs **`docs_dir` to `docs2`** (or equivalent) and either copy or symlink `stylesheets/` and `assets/` from `docs/` if they remain there, since theme paths (`extra_css`, `logo`) are resolved relative to the documentation root.
