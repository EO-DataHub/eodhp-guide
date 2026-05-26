---
title: 3.6 Workspace Storage
doc_status: ok
last_reviewed:
reviewed_by:
review_notes:
tags:
  - workspaces
  - aws
---
### 3.6 Workspace Storage

Workspaces may contain some number of private object and block stores. The architecture supports any number of these in each workspace, but the current implementation requires one of each type. Later extensions may allow multiple object and block stores so that different types may be offered, for example reduced-redundancy object stores or Lustre based block stores. 

Both store types are fully elastic and a particular storage size does not need to be allocated in advance. 

#### 3.6.1 Object Stores

Due to AWS bucket limits at the time of implementation, there is a single shared object store for workspace object storage and a separate S3 access point for each workspace store is layered on top of it. These access points restrict access to authorized users (and to a prefix in the shared bucket which is dedicated to a specific workspace) and are created by the workspace controller during workspace creation. 

Access using the S3 protocol is possible for workspace members by retrieving a temporary AWS STS credential from the workspace UI or API and using it directly with AWS’s service. This is possible both from within EODH services and externally across the Internet. 

#### 3.6.2 Block Stores

Workspace block stores are a share of an AWS EFS (NFS) filesystem. EFS access points are created for each workspace store, restricting access to a particular subdirectory. Direct access is only possible from within the EODH Kubernetes cluster, where the store can be mounted (such as in notebooks). However, the Data Access Services can provide HTTPS based access. 

AWS imposes various limits requiring a strategy for later scaling should the number of workspace stores reach 10,000: 

- 10,000 access points per EFS filesystem, 
- 1400 EFS mount targets per VPC (increased from 400 since the last version of this document), 
- 1000 EFS filesystems per region, 

To later scale the number of block stores, the Workspace Controller can begin to use a pool of filesystems instead of a single one, creating AWS EFS access points as before. This allows over a million stores to be allocated as well as scaling the per-filesystem throughput limits such as the 3GBps read throughput limit. 

Multiple pools of filesystems may be required if alternative filesystem options are provided, with one for each set of supported filesystem options (multi- vs single-zone, lifecycle management, provisioned throughput, etc). 

Support for AWS EFx Lustre filesystems could also be added later in the same manner. Whilst Lustre is designed for HPC applications and can be linked to an S3 bucket, so that the bucket contents appear in a sub-path, it also has some disadvantages. Access control is exclusively through POSIX UIDs/GIDs and storage must be allocated in advance in 2.4GB increments.
