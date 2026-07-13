---
title: Reusable CI workflows
doc_status: ok
tags:
  - ci-cd
  - github-actions
last_reviewed: 2026-06-29
reviewed_by: recmanj
review_notes:
---

# Reusable CI workflows

The [`github-actions`](https://github.com/EO-DataHub/github-actions) repository is the EO Data Hub's central store of **reusable GitHub Actions** — `workflow_call` workflows plus a composite action — shared across EODH repositories. A calling repository references them by path and ref:

```yaml
jobs:
  security-scan:
    permissions:
      contents: read
    uses: EO-DataHub/github-actions/.github/workflows/security.yaml@main

  qa:
    uses: EO-DataHub/github-actions/.github/workflows/qa-python.yaml@main

  unit-tests:
    permissions:
      contents: read
    uses: EO-DataHub/github-actions/.github/workflows/unit-tests-python-uv.yaml@main
    with:
      PYTHON_VERSION: "3.12"
```

For the shared Renovate dependency-update preset in the same repository, see [Shared Renovate configuration](renovate-config.md).

## Active workflows

All workflows live under `.github/workflows/` and are triggered via `workflow_call`.

| Workflow | Purpose |
|----------|---------|
| `ecr-publish.yaml` | Build and push a Docker image to AWS **public** ECR. |
| `s3-publish.yaml` | Sync a build-artifact directory to an S3 bucket. |
| `security.yaml` | Security scan: Trivy filesystem scan + Semgrep SAST. Also runs on `pull_request`. |
| `qa-python.yaml` | Python QA via `uv`: Ruff lint, Ruff format check, Pyright. |
| `unit-tests-python-uv.yaml` | Python unit tests with `uv` + pytest. |
| `unit-tests-python.yaml` | Python unit tests with pip + pytest. |
| `unit-tests-go.yaml` | Go unit tests (`go test -cover ./...`). |
| `go-build.yaml` | Go lint (golangci-lint) + tests with coverage gates. |
| `pre-commit-go.yaml` | Run `pre-commit` hooks (Go toolchain). |
| `pre-commit-node.yaml` | Run `pre-commit` hooks (Node LTS). |

Third-party actions used inside these workflows are pinned to commit SHAs (enforced by the Renovate `helpers:pinGitHubActionDigests` preset).

## Deprecated workflows

These remain for backwards compatibility but should not be used in new pipelines.

| Workflow | Replaced by | Notes |
|----------|-------------|-------|
| `docker-image-to-aws-ecr.yaml` | `ecr-publish.yaml` | Uses **secret-based** AWS auth (`AWS_REGION`, `AWS_ECR`, `AWS_ACCOUNT_ID`) rather than OIDC role assumption. |
| `get-version-tag.yaml` | `ecr-publish.yaml` | Tag derivation is now handled inside `ecr-publish`. |
| `get-docker-tag.yaml` | `get-version-tag.yaml` | Earliest Docker-tag helper. |

## Composite action: `storage-optimizer`

`.github/actions/storage-optimizer` is a composite action that frees disk space on the GitHub-hosted runner before a large build, removing pre-installed toolchains that EODH builds don't need (JDKs, .NET, Swift, Haskell, Julia, the Android SDK, Chromium, Azure CLI, PowerShell, and the cached `hostedtoolcache`). It is invoked automatically by `ecr-publish.yaml` when `cleanup: true`.

## Shared lint and pre-commit configs

The repository root also holds shared linter and `pre-commit` configuration that consuming repositories can adopt:

- `.pre-commit-config-go.yaml`, `.pre-commit-config-node.yaml`, `.pre-commit-config-python.yaml` — language-specific `pre-commit` hook sets.
- `.golangci.yml` — golangci-lint configuration (aligned with `go-build.yaml`).
- `.eslintrc.cjs`, `.stylelintrc.json` — JavaScript/CSS lint configuration for Node projects.
