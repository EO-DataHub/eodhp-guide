# EO DataHub Platform Deployment

## Infrastructure

The EODHP is designed to be deployed to AWS. The infrastructure deployment is managed by the [Terraform deployment](https://github.com/UKEODHP/eodhp-deploy-infrastucture).

The Terraform repo manages multiple deployment environments using workspaces. Before deploying ensure you are in the correct workspace.

The deployment configuration variables are stored in the terraform/envs/\*/\*.tfvars files.

```bash
cd terraform
terraform workspace list
terraform workspace select dev
terraform apply -var-file envs/dev/dev.tfvars
```

## Applications

The platform components and applications are managed using the GitOps framework ArgoCD in the [ArgoCD deployment](https://github.com/UKEODHP/eodhp-argocd-deployment).

The repo contains the configuration for multiple environments.

First ArgoCD has to be deployed to the cluster and then the ArgoCD application CRDs. Once ArgoCD applications have been applied then ArgoCD manages itself, as well as the platform applications.

```bash
kubectl apply -k apps/argocd
kubectl apply -k eodhp/envs/dev
```

## Manual Configuration

There are some manual steps required. While these will be automated as far as possible the current steps are outlined below.

### Keycloak

#### Update GitHub SSO configuration

In order for users to authenticate via GitHub, Keycloak needs access to the GitHub OAuth app secret.

##### First time setup

This step will not normally be required for restarting the dev or test clusters.

1. Log in to GitHub and navigate to the `KeyCloakAuth` OAuth App
2. Under `General`, navigate to `Client secrets` and click `Generate a new client secret`. Copy the secret
3. In AWS Secrets Manager, find the `eodhp-<env>` secret
4. Click `Retrieve secrets value`
5. Click `Edit`
6. Ensure that there is a key called `keycloak.auth.githubSecret`. Add if not present
7. Paste the secret in the values field
8. Save

#### Updating GitHub secret

1. Log in to AWS Secrets Manager and find the `eodhp-<env>` secret
2. Click `Retrieve secrets value`
3. Copy the value for the `keycloak.auth.githubSecret` entry
4. Navigate to the Keycloak admin panel at `<env>.eodatahub.org.uk/keycloak/admin` and log in
5. Select the `eodhp` realm
6. Click `Identity Providers`
7. Click `github`
8. Under `Settings`, update the Client Secret with the value from AWS
9. Save

#### Add Admin User to EODHP Realm

1. In Keycloak admin UI:
   1. Select eodhp realm
   2. Users
   3. Add user
      1. Set name as 'admin'
      2. Create
      3. In admin user credentials, set password (not temporary)

### Application Hub

#### Update OAUTH Client Secret

The application hub OAUTH_CLIENT_SECRET is generated by Keycloak when it is installed. A new secret is generated each time Keycloak is reinstalled.

The client secret is not automatically propagated to the App Hub, this must be done manually.

1. In Keycloak admin UI:
   1. Select eodhp realm
   2. Clients
   3. application-hub
   4. Credentials
   5. Copy Client secret
2. In AWS console:
   1. Secrets Manager
   2. eodhp-dev
   3. Retrieve secret value
   4. Edit
   5. Update app-hub.OAUTH_CLIENT_SECRET with new client secret
3. In cluster CLI:
   1. Ensure secret has propagated to app-hub secret:
      1. `kubectl get secrets -n proc app-hub -o yaml`
      2. Copy base64 encoded secret OAUTH_CLIENT_SECRET
      3. `echo <secret_value> | base64 -d`
      4. Confirm secret has propagated, if not repeat steps 1-3 until secret has propagated
   2. Restart application hub
      1. `kubectl rollout restart -n proc deploy/application-hub-hub`
4. Test login through Application Hub

#### Add Users to jupyter-lab Group

In order for users to have access to Jupyter Lab, they must be added to the jupyter-lab group in the Application Hub admin interface. This requires admin privileges within the Application Hub.

1. Log into Application Hub as admin
2. Select Admin tab
3. Manage groups
4. Create jupyter-lab group
5. Add users to group