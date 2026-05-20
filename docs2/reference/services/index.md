---
title: Reference — Services
doc_status: unreviewed
---

# Services

One Markdown file per deployed **component** or integration point (reuse `docs/services/.template.md` when adding pages).

Migrate the whole directory:

- Source: `docs/services/*.md` in the repo root (legacy path until migration completes).

| Legacy | `docs2` page |
|--------|----------------|
| `docs/services/accounting.md` | [accounting.md](accounting.md) (migrated) |
| `docs/services/argo-workflows-events.md` | [argo-workflows-events.md](argo-workflows-events.md) (migrated) |
| `docs/services/auth-agent.md` | [auth-agent.md](auth-agent.md) (migrated) |
| `docs/services/data-adaptors.md` | [data-adaptors.md](data-adaptors.md) (migrated) |
| `docs/services/database.md` | [database.md](database.md) (migrated) |
| `docs/services/elk.md` | [elk.md](elk.md) (migrated) |
| `docs/services/external-secrets.md` | [external-secrets.md](external-secrets.md) (migrated) |
| `docs/services/grafana.md` | [grafana.md](grafana.md) (migrated) |
| `docs/services/jupyter.md` | [jupyter.md](jupyter.md) (migrated) |
| `docs/services/keycloak.md` | [keycloak.md](keycloak.md) (migrated) |
| `docs/services/linkerd.md` | [linkerd.md](linkerd.md) (migrated) |

After files live here:

- Prefer **tabular or list facts** at the top of each page
- Push long narrative rationale into **Explanation** (architecture or design-and-decisions) and link back
