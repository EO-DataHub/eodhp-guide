# Updating Kubernetes Cluster Version

## Purpose

The Kubernetes version is defined in the Terraform deployment repo. Kubernetes has a new version release approximately 3 times per year.

The following should be borne in mind:

- Newer versions of Kubernetes can bring bugfixes, optimisations and new features.
- Kubernetes try to maintain compatibility between +/- 2 versions. Clusters using a version more than 2 minor releases old may have compatibility issues.
- AWS only support new Kubernetes versions for 14 months, after which extended support lasts a further 12 months. Extended support costs more.

For these reasons, the platform should always aim to use the latest stable Kubernetes version.

## When to Use

Use this procedure when you wish to update the platform AWS EKS version.

## Operation

**Please read section [Cluster Preparation](#cluster-preparation) first before proceeding.**

1. Open [Terraform Deployment](https://github.com/EO-DataHub/eodhp-deploy-infrastucture.git) repo in your preferred IDE
2. In terminal, change directory into _terraform/_ dir (`cd terraform`)
3. In _envs/$ENV.tfvars_, file (e.g. _envs/prod.tfvars_), set Kubernetes version to desired version. Note that you can only increment by one minor version at a time.
4. Scale all node groups down. Use the AWS console or CLI to scale autoscaling node groups down to zero for min, max and desired. Autoscaling node groups can be found in EC2 > Auto Scaling groups (left sidebar) > use filter with cluster name to filter for your environment's auto scaling groups only. For each group, under details tab, edit min, max and desired to 0, temporarily. This stage can be tedious, in practical terms this is usually only required for the `services` node groups.
5. You need to tell Terraform to temporarily no longer ignore the `desired_size` on deployment. Comment out `aws_eks_node_group` `lifecycle.ignore changes` for `scaling_config[*].desired_size` for each node group in _terraform/nodes-\*.tf_ files. If you do not do this, you will get an error about not being able to set `max_size` lower than `desired_size` when applying configuration. Make sure you do not commit this change, it will be reverted.
6. Execute a `terraform apply -var-file envs/$ENV.tfvars` and check that the plan will update the EKS version of all node groups as expected. If plan accepted, type `yes` and hit enter to start the rollout.
7. Monitor the logs and ensure that the rollout is proceeding as anticipated.
8. Once the rollout is complete, you can check the Kubernetes version of the cluster by either:
   1. In AWS console, navigate to Elastic Kubernetes Service > $CLUSTER > Compute (tab). Check Node Groups table and check _AMI release version_ to see Kubernetes version (should be of the form X.Y.Z-$DATE, where X.Y is the Kubernetes version).
   2. Assuming your `kubectl` context is pointed to the cluster, use `kubectl version` and inspect the server version. `kubectl get nodes` will also show the Kubernetes version of each node (should be of the form X.Y.Z-$DATE, where X.Y is the Kubernetes version).
9. If operation was successful:
   1. Revert step 5.
   2. Commit the Kubernetes version update change in the _envs/$ENV.tfvars_ file to Git.

## Requirements

- Write access to Terraform deployment repo
- `kubectl` access to cluster
- (Optional) Access to AWS console

## Useful Information

- Always ensure you are in the correct Terraform workspace with `terraform workspace list` before applying updates. Change the workspace with `terraform workspace select $WORKSPACE`.
- The Terraform apply when updating the Kubernetes version can take a very long time (~20 minutes). During this time the platform will be unavailable.
- If the update to any node groups fails, scale the group down using step 4. in [Operation](#operation) and try again.
