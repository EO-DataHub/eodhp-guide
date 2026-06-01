---
title: 3.7 Jupyter
doc_status: ok
last_reviewed:
reviewed_by:
review_notes:
tags:
  - jupyter
  - notebooks
---
### 3.7 Jupyter

```kroki-plantuml
@startuml

actor User

package WSStorage as "Workspace Stores" {
	node S3 {
	}

	node "NFS Volume" as WNFS {
	}
}

node "User Workspace (K8s namespace)" as UserWorkspace {
	[JupyterLab]
	[DaskCluster] as Dask
	JupyterLab --> Dask : Creates
}

node "Jupyter (K8s namespace)" as JupyterHubNS {
  [JupyterHub]
  JupyterHub --> JupyterLab : Spawns
}

JupyterLab ---> WNFS : Mounts
JupyterLab ---> S3 : Can browse\n(Jupyter Contents Manager)

User --> JupyterHub : Manages JupyterLab instances
User ---> JupyterLab : Uses\nNotebooks
@enduml
```

**Figure 3-7 Jupyter**

The EOEPCA Application Hub provides JupyterHub for spawning JupyterLab notebooks and shell access as workspace services. This means that the notebooks run in workspace namespaces whilst the hub uses its own. This allows for correct accounting of resource use by notebooks. When a user spawns a notebook they must choose both the notebook image to use and also the particular workspace to run it in. 

The notebooks use the workspace’s primary block store as the home directory for the notebook user. A Jupyter plugin, the S3 contents manager, makes the workspace object store available as a Jupyter contents manager, allowing the management of files and notebooks there using the Jupyter UI. This requires an AWS Lambda which creates a .s3keep file inside each ‘directory’ created in the object store. S3 contents manager (and, for empty prefixes, also the AWS S3 UI) will not show prefixes as directories without this file.

#### Dask integration

Distributed compute is available to notebook users via the [Dask Kubernetes Operator](https://kubernetes.dask.org/en/latest/). Users can create a `KubeCluster` from within a notebook; the resulting scheduler and worker pods run in the same workspace namespace as the JupyterLab pod. This keeps resource accounting correct and ensures that Dask workers cannot access resources in other workspaces.

Each workspace namespace has a `ResourceQuota` that caps Dask resource consumption and limits workspaces to one active Dask cluster at a time. GPU-accelerated Dask workers are available to workspaces in the highest access tier, provisioned via dedicated EKS node groups that scale from zero.

See [Dask Integration](../../explanation/architecture/dask-integration.md) for configuration details, GPU access tiers, and decommissioning guidance.
