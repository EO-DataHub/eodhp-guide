---
title: Summary
doc_status: outdated
last_reviewed:
reviewed_by:
review_notes:
---
## Summary

We use the following clusters and pipeline steps:

- `dev`:
  - Cluster owned by and for the use of the development team and updated at any time.
  - Changes are made at any time by developers.
- `test`:
  - Cluster to demonstrate all of the features that have passed developer peer review.
  - Updated on each successful issue completion (after acceptance of PR).
  - Available to partners to test latest features, but not recommended for stable testing.
  - Not all features in test will be released in the next staging release, some will be withheld using Kustomize overlays until they are intended for release.
- `staging`:
  - Cluster for demonstrating releases proposed for deployment to `prod` so that customer approval may be gained.
  - Releases are made to `staging` as required from `main`.
  - Staging is considered a stable platform for partners to test and demo against.
  - When a candidate release to `prod` is considered, updates to `staging` will be frozen so that User Acceptance Tests (UATs) can be performed. If UAT is approved then release to `prod` will occur.
- `prod`
  - Releases are made to `prod` on approval and/or after successful UAT.

## Releases processes

(This will increase in detail as we understand exactly how to run tests, what approvals are wanted, etc)

The deployment of the platform is managed by various configuration repos that declaritively capture the configuration of the platform infrastrcuture, services and policies and store these configurations in Git. The configuration repos, at the time of writing, include:
- ArgoCD deployment repo - this repo manages the deployment of platform services using GitOps. It references service artifacts by version and manages the other Kubernetes resources required to support the service, e.g. deployments, services, ingresses etc. The service artifacts are managed in separate code repositories with their own development and release processes. Fundamentally, each service repo release results in an versioned, immutable artifact (e.g. container image, static web artifact, Python package). These versioned artifacts are deployed through ArgoCD in a configuration controlled way using Git commits, so the full configuration of the platform is known for each commit.
- Terraform infrastructure repo - this repo manages the initialisation of each environment specific infrastructure resource. It also deploys some AWS and Kubernetes resources where it was not possible/practical to manage the deployment through ArgoCD. The configuration is applied using Terraform.
- Terraform supporting infrastructure repo - Similar to the Terraform infrastructure repo, but it manages resources shared among environments. It is deployed at the AWS account instance level.
- Open Policy Agent - Manages the open policy agent policies for each environment. The Git repository itself is referenced by each environment's Open Policy Agent service.

Mainly, platform releases are controlled by the ArgoCD deployment repo, with occasional updates to the infrastructure and policy repos, as required. The following describes the release process for the ArgoCD deployment repository.

Releases follow these sequential steps:

- Development:
  - Feature branches are pushed to `dev` branch for testing. The `dev` environment points to the `dev` branch.
  - Developers are free to push to `dev` at will without requirement for peer review.
  - New features shall normally apply to all env overlays in the ArgoCD repo, however features can be "withheld" by not including them in `env/staging` or `env/prod` if their release is to be deferred.
  - These are released to the `dev` cluster immediately after commit.
  - Release notes should be prepared for each feature in the platform release notes repo. This would be the `test` release.
  - During development, service artifacts my be referenced by release candidate tags, e.g. v.1.2.3-rc4. Before creation of PR only stable version tags should be used, e.g. v.1.2.3.
  - Before features can be completed they must be peer reviewed by Pull Request by appropriate peer. The release notes for the feature should also be reviewed.
  - After PR acceptance the feature branch(es) can be merged to `main`.
  - Software repositories are tagged when merged to `main` and a new build outputs are pushed to the repositories.
- Test:
  - Completed features are committed to the `main` branches of software repositories. The `test` cluster points to the `main` branch.
  - All integration tests should run on `test`.
- Staging:
  - Releases from `main` to `staging` can be performed as required. Multiple releases to `staging` may occur before a release to `prod`.
  - A PR should be created and reviewed by all devs before release.
  - The `staging` release notes should be updated with all features to be released. These can be copied over from `test` release notes.
  - On PR acceptance the merge can be actioned and partners will be notified and directed to the `staging` release notes.
  - At some point the customer will be asked to perform UAT. At this point no further releases will be made to `staging` until UAT is complete.
  - All integration tests should run on `staging`.
  - Hotfixes to `staging` should be merged back into `main`.
- Production:
  - Releases from `staging` to `prod` are made on approval and/or after UAT.
  - A PR should be created and reviewed by all devs before release.
  - No new tagging or building occurs.
  - `prod` released notes should be updated in line with `staging`.
  - Only smoke tests should run on `prod`.
  - On release to `prod` partners will be notified and directed to the `prod` release notes.
  - Hotfixes to `prod` should be merged back into `main` and `staging`.

```mermaid
flowchart LR
  dev["`dev` branch → dev cluster"]
  main["`main` branch → test cluster"]
  staging["`staging` branch → staging"]
  prod["`prod` branch → prod"]
  dev -->|"PR merge"| main
  main -->|"release PR"| staging
  staging -->|"UAT / approval"| prod
```

_Git strategy overview (feature flow through branches and environments)._
