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

!!! warning "Not live yet"
    This page describes the setup in the draft branches `publish-environments` (`github-actions`) and `ci-roles-by-environment` (`eodhp-deploy-supporting-infrastructure`). Until both are merged and applied, repositories publish with `GitHubAccessEODH`. Whether release builds should also push to `dev/<image>` is not agreed yet.

Images go to ECR Public (`public.ecr.aws/eodh`) through `ecr-publish.yaml`, and static sites to the `static-web-artefacts-eodh` bucket through `s3-publish.yaml`, both in [`github-actions`](https://github.com/EO-DataHub/github-actions). Each publishing repository has its own roles in the prod account (`235494775127`), defined in `terraform/github-publish-roles.tf` of `eodhp-deploy-supporting-infrastructure`.

## How publishing works

`ecr-publish.yaml` decides per run:

| Run | Environment | Role | Pushes to |
|---|---|---|---|
| Any run on the default branch or a tag, except pull requests | `release` | `gha-release-<repo>`, from the `aws_role_arn` input | `eodh/<image>` and `eodh/dev/<image>` |
| Any run on another branch, except pull requests | `dev` | `gha-dev-<repo>`, from the `dev` environment variable `AWS_DEV_ROLE_ARN` | `eodh/dev/<image>` |
| Pull request | none | none | nothing, build only |

Release images are tagged `latest` from the default branch, `1.2.3` from tag `v1.2.3`, and with the tag name for other tags. The `image_tag` input replaces all of these. Dev images are tagged `<branch>` and `<branch>-<short sha>`.

`s3-publish.yaml` always runs in `release` with `gha-release-<repo>`.

| Role | Can be assumed by | Can write |
|---|---|---|
| `gha-release-<repo>` | a job in the `release` environment of `EO-DataHub/<repo>` running `ecr-publish.yaml` from `github-actions` `main` (`s3-publish.yaml` for static sites) | the repo's own images in `eodh/<image>` and `eodh/dev/<image>`, or the repo's own prefixes in `static-web-artefacts-eodh` |
| `gha-dev-<repo>` | the same, from the `dev` environment | `eodh/dev/<image>` only |

The roles check the token subject. With the repository's subject customised (step 3 below), a release run of `accounting-service` sends:

```text
repo:EO-DataHub/accounting-service:environment:release:job_workflow_ref:EO-DataHub/github-actions/.github/workflows/ecr-publish.yaml@refs/heads/main
```

The ID form `repo:EO-DataHub@126242924/<repo>@<repo id>:...`, which GitHub sends for some runs, is trusted too.

Exceptions, from `github_publish_repos` in `terraform/envs/prod.tfvars`:

- `eodhp-guide`, `eodhp-workspace-ui`: S3 only, no `dev` role, trusted for `s3-publish.yaml`.
- `eodhp-rc-ui`: S3 prefix `static-apps/sg-rc-ui`, trusted from its `production`, `staging` and `test` environments, from any workflow.
- `ZOO-Project`: uses its own workflow, so its roles check the environment only.

Repositories with `job_workflow_refs = []`, like the last two, must keep the default OIDC subject and skip step 3 below.

Only the shared workflows on `github-actions` `main` can assume the roles, so anyone who can push to that branch controls what runs with every publish role. Branch and tag rulesets and CODEOWNERS are proposed but not in place.

## Add a publishing repository

You need admin on the GitHub repository and access to the prod account (AWS profile `eodhp-prod`). The examples use `repo=my-service` and image `my-service`.

1. In `eodhp-deploy-supporting-infrastructure`, add the repository to `github_publish_repos` in `terraform/envs/prod.tfvars`. `images` are the image names the repository pushes (its `IMAGE_NAME`):

    ```hcl
    "my-service" = { images = ["my-service"] }
    ```

    For a static site:

    ```hcl
    "my-site" = { s3_prefixes = ["static-apps/my-site"], job_workflow_refs = ["EO-DataHub/github-actions/.github/workflows/s3-publish.yaml@refs/heads/main"] }
    ```

    Apply it as the [README](https://github.com/EO-DataHub/eodhp-deploy-supporting-infrastructure#deploy) describes:

    ```bash
    make workspace-select WORKSPACE=prod
    make plan
    make apply
    ```

    This creates `gha-release-my-service`, `gha-dev-my-service` and the ECR Public repository `dev/my-service`. The release repository `my-service` is not in Terraform. If it does not exist, create it:

    ```bash
    aws ecr-public create-repository --profile eodhp-prod --region us-east-1 --repository-name my-service
    ```

2. Create the environments before anything calls the workflow. A job that names a missing environment creates it with no rules. Use the repository's default branch if it is not `main`, and add any other tag patterns it releases from (`eodh-jupyter-images` also has `default-*`, `eodh-jupyter-hub-*` and `python-*`):

    ```bash
    repo=my-service
    gh api -X PUT "repos/EO-DataHub/$repo/environments/release" --input - <<'JSON'
    {"deployment_branch_policy": {"protected_branches": false, "custom_branch_policies": true}}
    JSON
    gh api -X POST "repos/EO-DataHub/$repo/environments/release/deployment-branch-policies" -f type=branch -f name=main
    gh api -X POST "repos/EO-DataHub/$repo/environments/release/deployment-branch-policies" -f type=tag -f name='v*'

    gh api -X PUT "repos/EO-DataHub/$repo/environments/dev"
    gh variable set AWS_DEV_ROLE_ARN -R "EO-DataHub/$repo" -e dev -b "arn:aws:iam::235494775127:role/gha-dev-$repo"
    ```

    S3-only repositories do not need `dev`.

3. Add `job_workflow_ref` to the repository's OIDC subject:

    ```bash
    gh api -X PUT "repos/EO-DataHub/$repo/actions/oidc/customization/sub" --input - <<'JSON'
    {"use_default": false, "include_claim_keys": ["repo", "context", "job_workflow_ref"]}
    JSON
    ```

    This changes the subject for every AWS role the repository assumes. A role that matches the old subject exactly stops accepting it. `GitHubAccessEODH` matches on `repo:EO-DataHub/*` and is not affected.

4. Set the repository variables:

    ```bash
    gh variable set AWS_ROLE_ARN -R "EO-DataHub/$repo" -b "arn:aws:iam::235494775127:role/gha-release-$repo"
    gh variable set AWS_ECR_ALIAS -R "EO-DataHub/$repo" -b eodh
    gh variable set IMAGE_NAME -R "EO-DataHub/$repo" -b my-service
    ```

    For a repository that already publishes with `GitHubAccessEODH`, only `AWS_ROLE_ARN` changes. Change it last, after step 5.

5. Call the shared workflow at `@main`:

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

    A SHA or any other ref after `@` changes `job_workflow_ref`, and the roles refuse it.

## Publish a dev image

Push to a branch. Every push to a branch other than the default one publishes `public.ecr.aws/eodh/dev/<image>` with two tags: `<branch>`, which moves with each push, and `<branch>-<short sha>`, e.g. `my-branch-1a2b3c4`. Slashes in the branch name become dashes. Pull request runs only build. To list dev tags:

```bash
aws ecr-public describe-images --profile eodhp-prod --region us-east-1 --repository-name dev/<image> --query 'imageDetails[].imageTags' --output text
```

The Kargo `<app>-dev-images` warehouses still watch `public.ecr.aws/eodh/<image>`. Moving them to `dev/<image>` is not designed yet, so these images cannot be deployed to test through Kargo until it is.

## Troubleshooting

| Symptom | Cause |
|---|---|
| Publish job fails before it starts: `... is not allowed to deploy to release due to environment protection rules` | The tag or branch is not in the `release` environment's allowed refs. Add the tag pattern, or release from the default branch. |
| `Not authorized to perform sts:AssumeRoleWithWebIdentity` | The token subject does not match the role's trust. See [Refused role assumption](#refused-role-assumption). |
| Push fails because `dev/<image>` does not exist | Terraform has not been applied since the image was added. Release runs push to `dev/<image>` too, so they fail the same way. |
| Push denied on an `ecr-public:` action | `IMAGE_NAME` is not in the repository's `images` in `github_publish_repos`. |
| Pull request run builds but pushes nothing | Expected. The push to the branch publishes the dev image. |

### Refused role assumption

Usual causes:

- The repository's OIDC subject is not customised (step 3), or is customised for a repository with `job_workflow_refs = []`.
- The caller uses the shared workflow at a SHA or another ref instead of `@main`.
- `AWS_ROLE_ARN` or `AWS_DEV_ROLE_ARN` names another repository's role, or a role that does not exist because the repository is missing from `github_publish_repos` or Terraform has not been applied.
- `AWS_DEV_ROLE_ARN` is not set on the `dev` environment. A dev run then falls back to `AWS_ROLE_ARN`, and the release role refuses the `dev` environment.
- A repository with its own workflow runs the job outside the environments its roles trust.

To check the subject, compare what the repository sends with what the role trusts. Configured claims (expect `"use_default": false` and the three keys from step 3):

```bash
gh api "repos/EO-DataHub/$repo/actions/oidc/customization/sub"
```

Recent role assumptions, refused ones included, with the subject GitHub sent (image publishing assumes roles in `us-east-1`; S3 publishing uses the `aws_region` input):

```bash
aws cloudtrail lookup-events --profile eodhp-prod --region us-east-1 \
  --lookup-attributes AttributeKey=EventName,AttributeValue=AssumeRoleWithWebIdentity \
  --max-results 50 --query 'Events[].CloudTrailEvent' --output json \
  | jq -r '.[] | fromjson | [.eventTime, (.errorCode // "ok"), .requestParameters.roleArn, .userIdentity.userName] | @tsv'
```

What the role trusts:

```bash
aws iam get-role --profile eodhp-prod --role-name "gha-release-$repo" --query Role.AssumeRolePolicyDocument
```

## Where each setting lives

Everything except the roles and dev repositories is set by hand and is not in any repository.

| Setting | Where | Check with | Who can change it |
|---|---|---|---|
| `release` and `dev` environments, `release` allowed branches and tags | Repository settings, Environments | `gh api repos/EO-DataHub/<repo>/environments/release/deployment-branch-policies` | Repository admins |
| `AWS_DEV_ROLE_ARN` | `dev` environment variable | `gh variable list -R EO-DataHub/<repo> -e dev` | Repository admins |
| `AWS_ROLE_ARN`, `AWS_ECR_ALIAS`, `IMAGE_NAME` | Repository variables | `gh variable list -R EO-DataHub/<repo>` | Repository admins |
| OIDC subject claims | Repository Actions settings, API only | `gh api repos/EO-DataHub/<repo>/actions/oidc/customization/sub` | Repository admins |
| Release ECR Public repositories | Prod account, us-east-1, created by hand | `aws ecr-public describe-repositories --profile eodhp-prod --region us-east-1` | Prod account users |
| Roles, their trust, `dev/` repositories | `github_publish_repos`, `eodhp-deploy-supporting-infrastructure` | `aws iam get-role --profile eodhp-prod --role-name gha-release-<repo>` | Whoever applies that Terraform |
