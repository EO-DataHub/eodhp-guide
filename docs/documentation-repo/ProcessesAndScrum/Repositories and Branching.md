---
title: Software Repositories
doc_status: outdated
last_reviewed:
reviewed_by:
review_notes:
---
We use one repository per component (not a monorepo). Two types of repository are defined, with different branching strategies.


## Software Repositories

Software repositories are those which build an individual versioned and installable artefact, including:
* Software packages
* Docker images
* Webpack bundles

### Software Repository Branching

TODO: Consider how individual developers work with Kubernetes and whether `main` will become too unstable for generating releases.

Use:
* `main` as the branch to which code is merged when it's ready to deploy to the `dev` cluster. Ideally this is completed code but at least shouldn't break for other developers.
* `feature/EODHP-xxx-description` for work in progress for story EODHP-xxx.
* tags with semantic versions to generate a release to be installed on the `test` cluster and those further downstream.

The branching strategy is intended to be simple and may change as we begin to support a production-like system by introducing a separate `develop` branch.
### Software Repository Expected Contents

TODO: Create template repositories.

* A LICENSE file containing the Apache 2.0 license (unless the repo will be private).
* Following https://www.apache.org/licenses/LICENSE-2.0.html#apply, each file should contain (where true, of course)
    Copyright 2024 Telespazio UK Ltd
    
    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at
    
        http://www.apache.org/licenses/LICENSE-2.0
    
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

* A README.md file (and a `docs` directory with more files if the content needs to be broken up) containing at least:
	* A description of what the component is for.
	* How to build, test and package it.
	* How to set up a development environment (eg, how to install dependencies, how to run tests and the software itself in such an environment, how to configure autoformatters like `black` into an IDE).
	* How to check for known vulnerabilities in dependencies (eg, `pip-audit` or `npm audit`) and how to updated dependencies.
* Automated dependency management must be used, with specific versions specified. This should follow language-ecosystem recommendations and lock specific versions, eg
	* `package.json` and `package-lock.json` for JavaScript+npm
	* `pyproject.toml` in Python, with `requirements.txt` and `requirements-dev.txt` (if necessary) generated using `pip-compile` and `pip-compile --extra dev -o requirements-dev.txt`
	* the result of `go mod init` for `go`
* Configuration for linting and (preferably automated) formatting, using tools like `black` and `ruff` for Python and `gofmt` for Go.
* Consider a `Makefile` with targets `dockerbuild`, `dockerpush`, `test`, `lint` and similar.
* Github Actions configuration (TODO: template this) which will run on commit and on tag to generate a build. These should:
	* Fail on lint error or test error.
	* Fail if any vulnerable dependencies are detected.
	* Build a package which 1) for commits to branches is versioned with the branch name and a timestamp, or 2) for tags has a version number derived from the tag. A package would normally be a Docker image or, possibly, a Webpack bundle.
	* Push the package to a repository. This will probably be a Docker repository or, for web apps served from public buckets, S3.


## Configuration Control Repositories

Config control repositories control cluster state via the tools which manage them, including:
* Terraform
* ArgoCD
### Branching for Configuration Control Repositories

NOTE: This reflects my experience with ArgoCD (and Puppet) but not Terraform.

A single branch should be used, `main`, for controlling all clusters. The tool's own mechanism for overlaying configuration changes on a base configuration should be used to manage differences between environments. This should separate differences due to different development stage from changes due to different clusters (as is done in AI4DTE's `argocd-deployment` repo with its `base`/`targets`/`stages`/`envs` separation, except that `targets` is not necessary)
