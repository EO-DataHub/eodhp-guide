# Database

## Summary

This service creates postgres databases and configures them with users to allow other services to store data as required.

### Code Repositories and Artifacts

The deployment for the database service is configured in https://github.com/EO-DataHub/eodhp-argocd-deployment repository, apps/database.

The database service consists of a controller which is deployed via helm chart and configured using a values file:
- Helm chart defined in https://github.com/EO-DataHub/eodhp-argocd-deployment/blob/main/apps/database/base/controller/kustomization.yaml
- Values file defined in https://github.com/EO-DataHub/eodhp-argocd-deployment/blob/main/apps/database/base/controller/values.yaml

Databases are then defined in the base directory, including configmaps to define start up scripts for each database, specifying usernames, roles and search_paths.


### Dependent Services

Many services rely on databases being created and maintained, so if this services fails, there a re a number of dependent services that may face issues:
- Workflow Runner - stores workflow details and request information, handled by the ADES
- Auth Agent
- Keycloak
- Web Presence - stores Wagtail contents
- Workspace - stores workspace details, including ownership and membership information
- Accounting - stores billing events from billing collectors 


## Operation

The Database deployment runs in the Kubernetes namespace `databases`.

Database configuration can be done using Postgres UI applications, such as pg-admin.

### Configuration

Database is configured in [ArgoCD Deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment) repository, apps/database directory.

### Control

To restart the controller service run `kubectl rollout restart -n databases deployment postgres-operator-ext-postgres-operator` for Kubernetes cluster or use ArgoCD UI to restart.

You should not delete any databases in this namespace, as this can cause data loss.

### Client Access
If you need to connect directly to the Postgres SQL database (e.g. for debugging or manual queries), you can do so using a number of different clients, e.g. PGAdmin. The database credentials 
are stored in a Kubernetes secret. 

To extract the credentials apply the following command within the cluster:
```bash 
kubectl get secret {secret-name} -n databases -o jsonpath="{.data.POSTGRES_URL}" | base64 --decode
```

Where `{secret-name}` refers to one of the specific database secrets (e.g. workspaces). The different database secrets can be seen from:

```bash
kubectl get secrets -n databases 
```

### Dependencies

- No dependencies

### Backups

The databases created by this service are handled by AWS RDS, so backups are managed by AWS.

## Development

All code is stored and configured in  the deployment repository, [ArgoCD Deployment](https://github.com/EO-DataHub/eodhp-argocd-deployment), apps/database directory.
