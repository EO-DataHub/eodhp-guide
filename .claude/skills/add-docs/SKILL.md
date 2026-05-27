---
name: add-docs
description: Add a new documentation page to the EO Data Hub guide. Use when the user wants to write, create, add, or draft docs for this repo. Handles placement (Diátaxis), frontmatter, style matching, and .pages registration.
---

# Add a doc page

## What this skill does

1. Determines the right `docs/` path using the Diátaxis framework.
2. Creates the file with correct frontmatter and a skeleton body that matches the repo's writing conventions.
3. Registers the new page in the relevant `.pages` file.

All content lives under `docs/`. Do not create files under any other top-level directory.

---

## Step 0 — Understand the request

Ask (or infer from context):
- **What is this page about?** (topic, component, task, concept)
- **Who is the reader and what do they want when they open it?**

---

## Step 1 — Choose the quadrant

Use this decision tree:

| Reader intent | Quadrant | Root folder |
|---|---|---|
| Complete a specific task ("deploy X", "restart Y", "configure Z") | How-to | `docs/how-to/` |
| Look up a fact, catalogue, list, inventory, port, repo | Reference | `docs/reference/` |
| Understand why/how something works — rationale, design trade-offs | Explanation | `docs/explanation/` |
| Understand the system's architectural components and their relationships | Architecture | `docs/architecture/` |

If the content mixes modes, **split it** rather than blending. Link the parts together with cross-references.

---

## Step 2 — Pick the subfolder

### `docs/how-to/` subfolders

| Subfolder | Put here |
|---|---|
| `deployments/` | Terraform, AWS rollout, bootstrap prerequisites |
| `kubernetes-gitops/` | Argo CD, Kargo, cluster maintenance |
| `observability-logging/` | Grafana, Prometheus, ELK/Kibana tasks |
| `data-and-catalogues/` | Titiler/STAC ingestion, catalogue corrections |
| `identity-access/` | Keycloak bootstrap, user elevation, federation |
| `notebooks-and-workspaces/` | Notebook releases, dev workspace onboarding |
| `analytics/` | Analytics and reporting tasks |
| `documentation/` | Docs process and tooling guides |

### `docs/reference/` subfolders

Check `docs/reference/` for existing subfolders (`apis/`, `data/`, `services/`) and place content in the closest match, or at the root if none fits.

### `docs/explanation/` subfolders

Existing subfolders: `architecture/`, `development-and-governance/`, `iam/`. Create a new subfolder only if the content forms a coherent new topic cluster.

### `docs/architecture/`

Use for architectural-design diagrams and narrative overviews of system components.

---

## Step 3 — Create the file

### Filename

Lowercase, hyphen-separated, no spaces: `my-topic-name.md`.

### Frontmatter

Every page must start with this frontmatter block:

```yaml
---
title: <Human-readable title>
doc_status: unreviewed
tags:
  - <relevant-tag>
last_reviewed:
reviewed_by:
review_notes:
---
```

`doc_status` values: `unreviewed`, `ok`, `needs-update`, `outdated`, `needs-verification`.

### Style conventions

Match what the existing docs do:

- **How-to pages**: Start with any required tools/prerequisites, then numbered or headed steps. Use bash code blocks for commands. Include a note at the top if instructions assume a specific environment (e.g. `prod` workspace).
- **Reference pages**: Start with a one-paragraph summary of what this catalogue/inventory covers. Use tables for structured data. Bullet lists for short enumerations.
- **Explanation pages**: Start with a sentence stating *what* this page explains. Link to the related How-to and Reference pages near the top using the bold-label pattern: `**How-to:** [link] · **Reference:** [link]`. Use H2 headings for major concepts.
- **Architecture pages**: Use H2 headings per component or layer. Embed PlantUML or Mermaid diagrams where helpful.

### Index pages

If you are creating a new subfolder, also create an `index.md` for it following the pattern in `docs/explanation/iam/index.md`:
- Short intro sentence.
- Bold cross-links to related How-to and Reference sections.
- Bullet list of pages in this section.

---

## Step 4 — Register in `.pages`

Every directory with custom navigation has a `.pages` file. After creating the file:

1. Open the `.pages` file in the same directory.
2. Add a new entry under `nav:` in the appropriate position (alphabetical or logical order).

Format:
```yaml
nav:
  - Existing Page: existing-page.md
  - New Page Title: new-page.md     # ← add this
```

If no `.pages` file exists in the directory, create one:
```yaml
nav:
  - index.md
  - New Page Title: new-page.md
```

---

## Step 5 — Cross-reference

After creating the page, check whether any existing index or closely related page should link to it. If so, add a link there too.

---

## Quick example

User: "Add a how-to for rotating Keycloak client secrets."

1. **Quadrant**: How-to (concrete operator task).
2. **Subfolder**: `docs/how-to/identity-access/`.
3. **Filename**: `rotate-keycloak-client-secrets.md`.
4. **Frontmatter**: `title: Rotate Keycloak client secrets`, `doc_status: unreviewed`, tags `keycloak`, `identity-access`.
5. **Body**: Prerequisites section (tools), then headed steps.
6. **`.pages`**: Add entry to `docs/how-to/identity-access/.pages`.
7. **Cross-ref**: Add link in `docs/how-to/identity-access/index.md`.
