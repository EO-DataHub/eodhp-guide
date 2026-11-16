---
title: Publish from GitHub Actions to AWS
doc_status: unreviewed
tags:
  - ci-cd
  - github-actions
  - aws
last_reviewed:
reviewed_by:
review_notes:
---

# Publish from GitHub Actions to AWS

Images go to ECR Public (`public.ecr.aws/eodh`) through `ecr-publish.yaml`, and static sites to S3 through `s3-publish.yaml`, both in [`github-actions`](https://github.com/EO-DataHub/github-actions). Each publishing repository has its own AWS roles for these workflows.

## How publishing works

`ecr-publish.yaml` decides per run:

| Run | GitHub environment | Pushes to |
|---|---|---|
| Default branch or a tag | `release` | `eodh/<image>` and `eodh/dev/<image>` |
| Any other branch | `dev` | `eodh/dev/<image>` |
| Pull request | none | nothing, build only |

Release images are tagged `latest` from the default branch and `1.2.3` from tag `v1.2.3`. A tag that does not start with `v` is used as it is. A tag that starts with `v` but is not a version gets no image tag, so pass `image_tag`. The `image_tag` input replaces all of these. Dev images are tagged `<branch>` and `<branch>-<short sha>`.

`s3-publish.yaml` always runs in `release`.

The `release` environment accepts the default branch and the repository's release tags, `v*` unless more were configured. Other branches publish dev images only.

## Add a publishing repository

The roles, the GitHub environments and the repository variables are set up by the platform team. The steps are in `docs/github-publish-roles.md` of the private `eodhp-deploy-supporting-infrastructure` repository. Ask the platform team if you do not have access.

Once that is done, call the shared workflow at `@main`:

```yaml
on:
  push:
    branches:
      - '**'
    tags:
      - 'v*'
  pull_request: ~

jobs:
  publish:
    uses: EO-DataHub/github-actions/.github/workflows/ecr-publish.yaml@main
    with:
      image_name: ${{ vars.IMAGE_NAME }}
      aws_role_arn: ${{ vars.AWS_ROLE_ARN }}
      aws_ecr_alias: ${{ vars.AWS_ECR_ALIAS }}
    permissions:
      id-token: write
      contents: read
```

Keep `@main`. With a SHA or any other ref the roles refuse the run.

For a static site, upload the built site as an artifact in an earlier job and pass its name:

```yaml
  publish:
    needs: build
    uses: EO-DataHub/github-actions/.github/workflows/s3-publish.yaml@main
    with:
      app_artifact: my-site
      s3_path: ${{ vars.S3_PATH }}
      aws_region: ${{ vars.AWS_REGION }}
      aws_role_arn: ${{ vars.AWS_ROLE_ARN }}
    permissions:
      id-token: write
      contents: read
```

## Publish a dev image

Push to a branch. Every push to a branch other than the default one publishes `public.ecr.aws/eodh/dev/<image>` with two tags: `<branch>`, which moves with each push, and `<branch>-<short sha>`, e.g. `my-branch-1a2b3c4`. Slashes in the branch name become dashes. Pull request runs only build.

Dev tags are listed at `https://gallery.ecr.aws/eodh/dev/<image>`.

The Kargo `<app>-dev-images` warehouses watch `public.ecr.aws/eodh/dev/<image>`. To deploy a dev image to test, see [Deploying a dev/feature branch image](../kubernetes-gitops/kargo-developer-guide.md#deploying-a-devfeature-branch-image).

## Troubleshooting

| Symptom | Cause |
|---|---|
| Publish job fails before it starts: `... is not allowed to deploy to release due to environment protection rules` | The tag or branch is not allowed in the `release` environment. Release from the default branch or a `v*` tag, or ask the platform team to add the tag pattern. |
| `Not authorized to perform sts:AssumeRoleWithWebIdentity` | The workflow is not called at `@main`, or the repository is not set up for publishing. |
| Dev run fails in **Check dev role** | The repository's `dev` environment is not set up. |
| Push denied, or the `dev/<image>` repository does not exist | The image name is not registered for the repository. |
| Pull request run builds but pushes nothing | Expected. The push to the branch publishes the dev image. |

Check first that the workflow is called at `@main` and that `image_name` is the repository's registered image. The environments, roles and image repositories are set up by the platform team; see `docs/github-publish-roles.md` in `eodhp-deploy-supporting-infrastructure`.
