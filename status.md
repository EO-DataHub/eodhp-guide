---
title: Documentation review status frontmatter
doc_status: unreviewed
last_reviewed:
reviewed_by:
review_notes:
---
# Documentation review status frontmatter

Use this frontmatter on every MkDocs page to track review progress.

Add a **`title`** for MkDocs navigation and search (normally the same as the first `#` heading, or derived from the file name).

## Default frontmatter for existing pages

```yaml
---
title: Page Title
doc_status: unreviewed
last_reviewed:
reviewed_by:
review_notes:
---
```

## Status values

Use one of the following values for `doc_status`.

| Status               | Meaning                                                                                         |
|----------------------|------------------------------------------------------------------------------------------------|
| `unreviewed`         | The page has not been checked yet. Use this as the default for migrated pages.                   |
| `ok`                 | The page has been reviewed and is broadly fine as-is.                                           |
| `needs-update`       | The page is mostly valid, but some content is stale or incomplete.                            |
| `outdated`           | The page is known to be wrong or significantly stale.                                         |
| `needs-expansion`    | The page is accurate but too thin—missing examples, context, screenshots, or detail.            |
| `needs-verification` | The page looks plausible, but needs confirmation against code, product behaviour, or team knowledge. |
| `deprecated`         | The page describes something no longer used, but may be kept for history.                      |
| `remove-candidate`   | The page should probably be deleted, merged, or redirected elsewhere.                        |

## Recommended workflow

Start every migrated page as:

```yaml
title: Page title
doc_status: unreviewed
```

After review, update it to one of:

- `doc_status: ok`
- `doc_status: needs-update`
- `doc_status: outdated`
- `doc_status: needs-expansion`
- `doc_status: needs-verification`
- `doc_status: deprecated`
- `doc_status: remove-candidate`

## Optional detailed tracking

Use separate fields for notes and actions rather than creating too many status values:

```yaml
---
title: Page Title
doc_status: needs-update
last_reviewed: 2026-05-19
reviewed_by: ddowding
review_notes: "CLI flags changed; examples need updating."
needs:
  - verify commands
  - add screenshots
---
```
