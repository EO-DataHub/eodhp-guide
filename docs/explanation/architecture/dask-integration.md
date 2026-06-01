---
title: Dask Integration
doc_status: unreviewed
tags:
  - jupyter
  - notebooks
last_reviewed:
reviewed_by:
review_notes:
---

# Dask Integration

Explains how distributed compute via Dask is integrated with JupyterHub on the EODH platform, including GPU support, resource controls, and decommissioning.

**Reference:** [JupyterHub service](../../reference/services/jupyter.md)

---

## Implementation: Dask Kubernetes Operator

The platform uses the [Dask Kubernetes Operator](https://kubernetes.dask.org/en/latest/) rather than Dask Gateway. Key reasons:

- Simpler to deploy (single Helm chart, no multi-component proxy/auth service).
- Adequate for the initial use case (individual workspace users running notebook-driven workloads).
- Multi-tenancy and access control are handled via standard Kubernetes RBAC and `ResourceQuota` objects rather than a separate auth layer.

The trade-off is that the Operator has no built-in user authentication or centralised management dashboard. This is acceptable because users are already authenticated to their workspace via JupyterHub, and all Dask work is initiated from within a notebook.

---

## How a user creates a Dask cluster

From a JupyterHub notebook, install `dask-kubernetes` (or use the pre-installed version in the workspace image) and create a cluster:

```python
from dask_kubernetes.operator import KubeCluster
from dask.distributed import Client

cluster = KubeCluster(
    name="my-cluster",
    namespace="ws-<workspace-name>",  # use the workspace namespace
    n_workers=4,
    resources={
        "requests": {"memory": "2Gi", "cpu": "1"},
        "limits":   {"memory": "4Gi", "cpu": "2"},
    },
)
client = Client(cluster)
```

**Always delete the cluster when finished** — see [Decommissioning](#decommissioning-a-dask-cluster) below.

---

## Resource quotas

Each workspace namespace has a `ResourceQuota` that caps what Dask (and other pods) can consume:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dask-workspace-quota
  namespace: ws-<workspace-name>
spec:
  hard:
    requests.cpu: "10"
    requests.memory: 20Gi
    limits.cpu: "20"
    limits.memory: 40Gi
    count/daskclusters.kubernetes.dask.org: "1"  # one cluster per workspace
    pods: "10"
```

A side-effect of `ResourceQuota` is that **all pods in the namespace must declare resource requests and limits**, including the JupyterHub single-user pod. Ensure the JupyterHub spawner configuration sets these on user pods.

---

## GPU support

### Access tiers

GPU access is provisioned per workspace in three tiers:

| Tier | Access |
|------|--------|
| A    | JupyterHub only |
| B    | JupyterHub + Dask (CPU) |
| C    | JupyterHub + Dask + GPU |

GPU access (Tier C) must be explicitly requested and enabled by the operations team.

### Node groups

GPU nodes are EKS node groups that scale from zero. They are tainted so no pod is scheduled on them without an explicit toleration:

```
nvidia.com/gpu=<value>:NoSchedule
```

Node types provisioned:

| Environment | Single GPU | Multi-GPU |
|-------------|-----------|-----------|
| Test/Staging | g4dn.xlarge (max 2) | g4dn.12xlarge (max 1) |
| Production  | g5.xlarge (max 4) | g5.12xlarge (max 2) |

Node scale-up adds approximately 2 minutes to cluster startup. Nodes scale back to zero automatically ~10 minutes after all GPU pods are removed.

The [NVIDIA Kubernetes device plugin](https://github.com/NVIDIA/k8s-device-plugin) is deployed to expose GPU resources to the scheduler.

### CUDA notebook images

A CUDA-enabled Jupyter image is maintained in [eodh-jupyter-images](https://github.com/EO-DataHub/eodh-jupyter-images). It extends the standard CPU image and includes:

- CUDA runtime
- TensorFlow
- (Other ML libraries as added)

Images follow the tag pattern `python-<version>-<date>-cuda` (e.g. `python-3.13-2026.03.0-cuda`).

### Creating a GPU Dask cluster from a notebook

Use the [RAPIDS](https://hub.docker.com/r/rapidsai/base/tags) image for workers, which provides CUDA-accelerated Dask:

```python
from dask_kubernetes.operator import KubeCluster
from dask.distributed import Client
from distributed.diagnostics.plugin import PipInstall

cluster = KubeCluster(
    name="gpu-cluster",
    namespace="ws-<workspace-name>",
    n_workers=2,
    image="rapidsai/base:26.04a-cuda12-py3.12",
    resources={
        "requests": {"memory": "2Gi", "cpu": "1"},
        "limits":   {"memory": "4Gi", "cpu": "2", "nvidia.com/gpu": "1"},
    },
)
client = Client(cluster)

# Install any extra packages on the workers
client.register_plugin(PipInstall(packages=["cupy"]))
```

> GPU node provisioning takes roughly 2 minutes. The cluster dashboard will show workers as pending until the node is ready.

**Dependency note:** The `dask`, `distributed`, and `dask_kubernetes` versions in the notebook environment must match those in the worker image. Mismatches cause silent failures. Check versions with:

```python
import dask, distributed, dask_kubernetes
print(dask.__version__, distributed.__version__, dask_kubernetes.__version__)
```

---

## Decommissioning a Dask cluster

GPU (and CPU) nodes only incur cost while pods are running. Users should explicitly delete their cluster when done:

```python
cluster.close()
```

Or from `kubectl`:

```bash
kubectl delete daskcluster <cluster-name> -n ws-<workspace-name>
```

After all pods are removed, GPU nodes scale to zero automatically (~10 minutes idle).

To check for any orphaned clusters across all namespaces:

```bash
kubectl get daskclusters --all-namespaces
```

---

## Disabling GPU access for a workspace

To block a workspace from creating GPU clusters (moving it from Tier C to Tier B), set the GPU resource quota to zero:

```bash
kubectl patch resourcequota dask-workspace-quota \
  -n ws-<workspace-name> \
  --type merge \
  --patch '{"spec": {"hard": {"limits.nvidia.com/gpu": "0"}}}'
```

This prevents new GPU pod scheduling without removing the workspace's Dask (CPU) capability.

---

## Known issues and future improvements

- **ResourceQuota and JupyterHub pods:** When a `ResourceQuota` is added to a workspace namespace, all pods must declare resource limits. If the JupyterHub spawner does not set limits on user pods, new server launches will be rejected. Ensure spawner configuration includes resource declarations.
- **Kyverno for GPU access control:** A Kyverno policy based on workspace namespace labels is the preferred long-term mechanism for controlling GPU access, rather than patching `ResourceQuota` objects manually.
- **Namespace isolation:** All Dask clusters are created in the user's workspace namespace, which is correct. Future work should verify that RBAC prevents cross-workspace cluster creation.
- **Dependency management:** Dask does not natively support installing user-defined packages on workers through the Operator. The `PipInstall` plugin works for this but is not officially supported in all configurations. A shared base image with common dependencies locked reduces this risk.
