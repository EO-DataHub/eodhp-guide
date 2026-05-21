---
title: Debug EKS nodes (SSH via EC2 Instance Connect)
doc_status: ok
tags:
  - aws
  - kubernetes
last_reviewed: 2024-05-20
reviewed_by: geodowd
review_notes: "From docs/Development.md (Debugging EKS Nodes)"
---

# Debug EKS nodes (SSH via EC2 Instance Connect)

Sometimes it is necessary to access the EKS node EC2 instances via SSH. Predominantly our nodes do not have public IP addresses, but we can connect via a VPC endpoint.

The VPC endpoint service is known as EC2 Instance Connect. The existing endpoints can be viewed through the AWS console by going to VPC > Endpoints (side panel under Virtual private cloud).

EC2 Instance Connect is deployed as part of the [eodhp-deploy-supporting-infrastructure](https://github.com/EO-DataHub/eodhp-deploy-supporting-infrastructure) repo, and the security groups for each cluster are configured in the [eodhp-deploy-infrastucture](https://github.com/EO-DataHub/eodhp-deploy-infrastucture) repo. See also [Deployment repositories](../../reference/deployment-repositories.md).

The AWS CLI is required to connect to node instances in a private subnet.

## SSH into a node

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
