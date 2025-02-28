# EODHP Development Guide

## Sprint Cycle

### ArgoCD Deployment

The configuration of each environment is controller by the `eodhp-argocd-deployment` repo. The deployment is managed by ArgoCD in a GitOps methodology. Each deployment environment is linked to a specific branch of the `eodhp-argocd-deployment` repo and merges to these branches should be controlled.

The environments are linked to the following branches:

- dev -> dev-XX-Y
- test -> main
- staging -> staging
- prod -> prod

The flow of new features will follow this path:

> feature branch -> dev-XX-Y -> main -> staging -> prod

The following approvals are required for branch merges:

- feature branch -> dev-XX-Y: No approval needed
- dev-XX-Y -> main: PR required (any peer)
- main -> staging: PR required (all devs)
- staging -> prod: PR required (all devs)

At the start of each sprint a fresh branch is taken from `main` and named `dev-XX-Y` where XX is the sprint number and Y is the iteration that sprint. Y is necessary as sometimes it may be necessary to refresh the dev branch multiple times during a sprint.

When a developer begins work on an issue they take a branch from `main`. The branch should follow Gitflow naming and mention the Jira issue key, e.g. `feature/EODHP-123-my-new-feature`. Work on code updates on the feature branch, and when ready to test merge into current develop branch. Repeat this process until the issue is resolved. At this point, create a PR and assing reviewer(s). Once PR has been accepted merge completed feature back into main.

`main` branch is always the source of truth for the latest configuration of the EODH deployment.

### Applications / Packages

All other code repositories, including the Terraform infrastructure deployment repos, follow a more conventional GithubFlow as follows:

- feature branch -> main: PR required (any peer)
- release from main using github actions (must include git tagging of release commit)

The updated release can then be included in the ArgoCD deployment repo following the methodology above.

## Deploying to a Development Cluster

Code on a feature branch in a code repository can be deployed to a development cluster. It's assumed here that the repository is using the shared GitHub actions in https://github.com/EO-DataHub/github-actions

First, push your changes to GitHub on the feature branch, for example `feature/EODHP-123-example`. The GitHub actions will run to lint ('pre-commit'), security scan, unit test and build a Docker image ('aws-ecr-build'). Assuming aws-ecr-build runs successfully, a Docker image with a tag such as `feature-EODHP-123-example-latest` will be pushed to AWS ECR.

Secondly, edit the ArgoCD configuration for the development cluster and change the image for the component to use the new tag, `feature-EODHP-123-example-latest`. This shouldn't be merged to branches controlling other clusters so this doesn't need to be part of a PR or ArgoCD feature branch. The target service also needs to use `imagePullPolicy: Always`, which can be set and merged as part of the later PR if it isn't set already.

Finally, either:

- go to the ArgoCD UI, find the deployment for your app (marked as type 'deploy') and choose 'Restart' from its menu;
- OR run `kubectl rollout -n <namespace> restart deployment/<deployment-name>`.

The first and third steps can be repeated without needing to change ArgoCD.

Once you've finished, set the image tag to `latest` to tell other developers you're not using the development cluster to test this service and to set it back to images built from `main`.

## Debugging EKS Nodes

Sometimes it is necessary to access the EKS node EC2 instances via SSH. Predominantly our nodes do not have public IP addresses, but we can connect via a VPC endpoint.

The VPC endpoint service is known as EC2 Instance Connect. The existing endpoints can be viewed through the AWS console by going to VPC > Endpoints (side panel under Virtual private cloud).

EC2 Instance Connect is deployed as part of the eodhp-deploy-supporting-infrastructure repo, and the security groups for each cluster are configured in the eodhp-deploy-infrastucture repo.

The AWS CLI is required to connect to node instances in a private subnet.

### SSH into a Node

You must first push your own public SSH key to the node.

```bash
aws ec2-instance-connect send-ssh-public-key \
  --region eu-west-2 \
  --availability-zone eu-west-2a \
  --instance-id i-a12b3c4e5f6g7h8i9 \
  --instance-os-user ec2-user \
  --ssh-public-key file://~/.ssh/id_rsa.pub
```

Set the region, availability zone, instance-id, os-user and ssh public key as required. The instance ID of the node can be found through the AWS EC2 console.

Once you have pushed your SSH key you can then connect via SSH using the AWS CLI.

```bash
aws ec2-instance-connect ssh --instance-id i-a12b3c4e5f6g7h8i9
```
