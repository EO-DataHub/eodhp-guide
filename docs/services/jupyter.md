# Jupyter Notebooks

## Summary

The JupyterHub allows hub users to perform data analysis server side in the platform. JupyuterHub provides an web UI to start user notebooks. Notebooks are scoped to workspaces, allowing access to workspace data.

Users can choose from a range of notebook servers to perform their analysis in. Through the notebook they can access files in the workspace file and S3 storage through the Jupyter file browser.

### Code Repositories and Artifacts

- JupyterHub deployment configuration controlled in [ArgoCD Deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, _apps/jupyter_ directory.
- JupyterHub images, including custom Hub and user notebook images, controlled in [EODH Jupyter Images](https://github.com/EO-DataHub/eodh-jupyter-images) repo
- Custom plugins for JupyterHub (installed in EODH Jupyter Images repo) configuration controlled in [EODH JPyAuth](https://github.com/EO-DataHub/eodh-jpyauth) repo

### Dependent Services

- Jupyter notebooks will not be available through the web presence

## Operation

JupyterHub is a 3rd party open source software for allowing users to manage their own Jupyter notebooks as part of the EO DataHub platform. Access to users is through the web presence nav bar, or by navigating to https://eodatahub-workspaces.org.uk/notebooks.

From JupyterHub, users can start and stop workspace notebooks for all workspaces they are a member of. They will be redirected to any new notebooks, or can connect to any notebooks already running using the JupyterHub app.

JupyterHub runs as a deployment, `hub`, in Kubernetes in the `jupyter` namespace. User notebooks run in the relevant workspace namespaces, `ws-<workspace-name>`.

### Configuration

Jupyter is configured in [ArgoCD Deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, apps/jupyter directory.

You can check the currently running JupyterHub version using https://eodatahub-workspaces.org.uk/notebooks/hub/api and the current JupyterLab version using `conda list` in a JupyterLab terminal.

### Control

To restart service run `kubectl rollout restart -n jupyter deployment hub` for Kubernetes cluster or use ArgoCD UI to restart.

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

- Keycloak must be up to allow OIDC access to JupyterHub
- Nginx Ingress Controller must be working to access JupyterHub through the web UI

### Backups

Jupyter manages its own SQLite database as part of its deployment, which is in a self mounted Kubernetes volume. Even if this were to perish, no significant state is lost as user's are automatically added back into the database when they successfully log in through the EO DataHub IAM.

Workspace data is mounted from EFS and S3 stores and do not require to be backed up by Jupyter.

## Development

JupyterHub is an 3rd-party open-source project, https://github.com/jupyterhub/jupyterhub. Stock images are available at quay.io/jupyterhub/k8s-hub.

This project has developed custom hub image for Jupyter, which is managed at https://github.com/EO-DataHub/eodh-jupyter-images. This custom image is published to AWS ECR public.ecr.aws/eodh/eodh-jupyter-hub. The custom image includes a custom auth module to allow for workspace scoped Jupyter notebooks. New hub images can be created by released by following README.md in the repo for the _hub/_ image.

Updates to the custom JupyterHub plugins can be made in https://github.com/EO-DataHub/eodh-jpyauth repo and incorporated into new eodh-jupyter-images.
