## Terraform

The [Terraform CLI](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli), [kubectl]([Command line tool (kubectl) | Kubernetes](https://kubernetes.io/docs/reference/kubectl/)) and [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/) are required to deploy the infrastructure.

The Terraform deployments depend on AWS CLI profiles being configured for the respective target AWS account instances.

- profile `eodhp-test` for `dev` and `test` clusters
- profile `eodhp-prod` for `staging` and `prod` clusters
  To set up an AWS CLI profile use `aws configure --profile $PROFILE`. At all times you must ensure you use the correct `--profile $PROFILE` flag in all AWS CLI calls.

If you need to inspect outputs from a previous run of Terraform deployments use `terraform output`.

### AWS Shared Infrastructure

The following instructions are for the `https://github.com/EO-DataHub/eodhp-deploy-supporting-infrastructure.git` repository.

The AWS shared infrastructure is deployed once per AWS instance. It creates and shared resources between clusters, such as:

- VPC
- Internet gateways
- NATs
- Database server
- S3 buckets

```bash
# instructions are for prod workspace, update as necessary for other workspaces
cd terraform
terraform workspace list  # see available workspaces
terraform workspace new prod  # if workspace does not exist
terraform workspace select prod
terraform init  # if first time setting up local repo
# confirm input variables in envs/prod.tfvars file
terraform apply -var-file envs/prod.tfvars
# check plan and accept if everything ok
# it may be necessary to run more than once if the plan fails
# inspect the output variables to see key parameters
```

### Deployment Infrastructure

The following instructions are for the `https://github.com/EO-DataHub/eodhp-deploy-infrastucture.git` repository. Instructions are for prod workspace, update as necessary for other workspaces

The deployment infrastructure is deployed once per environment. It contains per environment resources such as:

- EKS cluster
- Cloudfront instances
- Route53 routes
- Cluster subnets
- Node groups and configuration
- IAM roles
- ECR pull-through cache
- SES (Simple Email Service)

Before starting, a secret must be created in AWS Secrets Manager with the name `ecr-pullthroughcache/dh` and the following values:

* `username`: a DockerHub username which will be used for cluster image pulls
* `accessToken`: a DockerHub personal access token

(Note: Terraform truncates the secret name when configuring ECR so 'dh' cannot become 'dockerhub')

```bash
cd terraform
terraform workspace list  # see available workspaces
terraform workspace new prod  # if workspace does not exist
terraform workspace select prod
terraform init  # if first time setting up local repo
# confirm input variables in envs/prod.tfvars file. Refer to output parameters from eodhp-deploy-supporting-infrastructure execution, where necessary.
terraform apply -var-file envs/prod.tfvars
# check plan and accept if everything ok
# it may be necessary to run more than once if the plan fails
```

### Kubectl Context

Create the `kubectl` context of the AWS EKS cluster. You will need the [aws-iam-authenticator]([kubernetes-sigs/aws-iam-authenticator: A tool to use AWS IAM credentials to authenticate to a Kubernetes cluster](https://github.com/kubernetes-sigs/aws-iam-authenticator)) to create the kubectl context.

```bash
aws --profile $PROFILE eks --region $REGION update-kubeconfig --name $CLUSTER_NAME --role-arn $ROLE_ARN --alias $ALIAS
```

Where:

- PROFILE is the profile you have set up for the AWS account instance you are targeting. See available profiles with `aws configure list-profiles`
- REGION can be found from Terraform output `region`
- CLUSTER_NAME can be found from Terraform output `cluster_name`
- ROLE_ARN can be found from Terraform output `eks_access_principal_arn`
- ALIAS is for your own reference, but suggest to use Terraform output `cluster_prefix`

```bash
kubectl config get-contexts
kubectl config use-context eodhp-prod
```

## ArgoCD

The ArgoCD configuration deploys all of the EODH services.

You will need [kubectl]([Command line tool (kubectl) | Kubernetes](https://kubernetes.io/docs/reference/kubectl/)), [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/), [Kustomize]([Kustomize - Kubernetes native configuration management](https://kustomize.io/)), [Helm]([Helm | Installing Helm](https://helm.sh/docs/intro/install/)) and [Gomplate]([gomplate - gomplate documentation](https://docs.gomplate.ca/)) to manage the ArgoCD deployment.

### Preparation

```bash
# https://github.com/EO-DataHub/eodhp-argocd-deployment.git
# instructions are for prod overlay, update as necessary for other overlays
# inspect eodhp/envs/prod/kustomization.yaml patches to see target branch
git checkout $TARGET_BRANCH
# check your kubectl context is pointed to the cluster set up in eodhp-deploy-infrastucture
kubectl config get-contexts
kubectl config set-context $CONTEXT  # if you need to update it
```

### Bootstrapping

If this is the first deployment of the environment then you must first bootstrap ArgoCD and then complete some initial setup. The bootstrap process exists because applications normally track `kargo/<app>/<env>` branches managed by Kargo, but on first deployment these branches don't exist yet.

1. Bootstrap ArgoCD

```bash
make bootstrap
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo  # get argocd initial password
kubectl port-forward service/argocd-server -n argocd 8080:443  # port forward argocd ui (necessary until platform deployment has created argocd ingress)
```

2. Visit http://127.0.0.1:8080 and log in to admin panel
3. Connect argocd to repository (not necessary if repo is public):
   Name: eodh
   Project: leave empty
   Repository URL: `git@github.com:EO-DataHub/eodhp-argocd-deployment.git`
   SSH private key data: Use repository deploy key
   Connect
   Status show successful connection.
4. Deploy applications in bootstrap mode. This tracks the `main` branch as a fallback since Kargo branches don't exist yet.

```bash
make deploy-bootstrap env=<env>
```

   You should see applications begin to initialise in the argocd UI. There may be race conditions as the apps come up. If any fail then enter the app in the UI and sync them again.

   Initially these are the core applications that should be up:

   - argocd
   - autoscaler
   - aws
   - cert-manager
   - certs
   - database
   - external-secrets
   - nginx
   - opal
   - redis
   - pulsar
   - replicator
   - secret-generator

   Apps that will not work until further configuration are, they can be ignored for now:

   - auth-agent
   - oauth2-proxy

   While the mentioned applications should be synced and healthy on their own at this stage, it has been observed that race conditions can occur that are too complex to diagnose. If this happens, you can try the following:

   - Force sync the app from the applications UI page
   - Try enabling server side apply when syncing app
   - Delete the app in the ArgoCD UI and see if it comes back healthy (which should happen automatically due to the ApplicationSet, but you may wish to refresh all apps in the UI to speed the recreation up).
   - If you are seeing errors regarding missing app namespaces you can create them manually with `kubectl create ns $NAMESPACE`.

   Once the core applications are synced and healthy you may proceed.

### Keycloak Configuration

Keycloak must be configured before Kargo promotions can be performed, as the Kargo UI authenticates via Keycloak OIDC.

5. Configure Keycloak admin user for access to ArgoCD UI via ingress.
6. Get keycloak initial admin password with `kubectl -n keycloak get secret keycloak-initial-admin -o jsonpath="{.data.password}" | base64 -d; echo`
7. Visit https://eodatahub.org.uk/keycloak and login to `temp-admin` account using initial password.
8. In the master realm, create an admin user with a password (under the credentials tab).
9. Give the admin user the `admin` realm role.
10. Log out of `temp-admin` and into `admin` and confirm that you have administration privileges. Once confirmed, you may delete `temp-admin` user.
11. In the master realm, create an ArgoCD OIDC client.
    Client ID: argocd
    Name: ArgoCD
    Root URL: https://argocd.eodatahub.org.uk
    Home URL: https://argocd.argocd.eodatahub.org.uk/applications
    Valid redirect URIs: /\*
    Web origins: +
    Client authentication: true
    Authorization: false
    Standard flow: true
    Direct access grants: false
12. Get the client secret from the client credentials tab and save it under the `keycloak.argocd.oauth2.secret` key in the AWS `eodhp` secret store. Wait a minute for the `external-secrets` controller to pull the password and restart the argocd server with `kubectl rollout restart -n argocd deploy/argocd-server`
13. Under the argocd client in Keycloak, visit Client scopes > argocd-dedicated > Add mapper > From predefined mappers > realm roles > Add
    Edit the newly created mapper to have the following settings:
    Name: realm roles
    Realm Role prefix: ""
    Multivalued: true
    Token Claim Name: roles
    Claim JSON Type: String
    Add to ID token: true
    Add to access token: false
    Add to lightweight access token: false
    Add to userinfo: false
    Add to token introspection: true
14. You should now be able to visit https://argocd.eodatahub.org.uk and sign in with keycloak for admin access to ArgoCD UI.
15. In the eodhp realm, create a Kargo OIDC client. Kargo uses Authorization Code Flow with PKCE and does not require a client secret.
    Client ID: kargo
    Name: Kargo
    Root URL: https://kargo.eodatahub.org.uk
    Valid redirect URIs: /\*
    Web origins: +
    Client authentication: false
    Authorization: false
    Standard flow: true
    Direct access grants: false
16. Ensure users who need Kargo admin access are assigned to the `admin` group in the eodhp realm.

### Kargo Promotion

With Keycloak configured, complete the bootstrap by promoting applications through Kargo.

17. Promote all applications via the [Kargo UI](https://kargo.eodatahub.org.uk). This creates the `kargo/<app>/<env>` branches for each application.

18. Redeploy to switch applications from the `main` branch fallback to the Kargo-managed branches.

```bash
make deploy env=<env>
```

### Kargo Integration

Applications use [Kargo](https://kargo.io) for progressive delivery. Each application tracks a dedicated branch in the format `kargo/<app-name>/<env>` (e.g., `kargo/accounting-service/test`). When Kargo promotes an application, it renders the manifests from `main`, commits them to the `kargo/<app>/<env>` branch, and ArgoCD syncs from that branch.

For full details on how Kargo and ArgoCD work together, warehouses, stages, promotion pipelines, and developer workflows, see the [Kargo Developer Guide](operations/kargo/developer-guide.md).

### Keycloak EODHP Realm Configuration

The Keycloak EODHP realm should have been configured on Keycloak initialisation. When Keycloak initialises the OIDC clients it will generate a client secret for each. These client secrets need to be copied into the AWS `eodhp` secret store so that ArgoCD can inject them into the relevant applications. The OIDC clients and their `eodhp` secret keys are given below:

- eodh - eodh.oidc.clientSecret
- eodh-workspaces - eodh-workspaces.oidc.clientSecret

Once these have been updated and you have waited a minute for `external-secrets` to sync, return to the ArgoCD UI and sync first the `oauth2-proxy` app and then the `auth-agent` app.

### Open Policy Agent

You need to ensure that an Open Policy Agent branch has been created for the cluster. The OPA repo is https://github.com/EO-DataHub/eodhp-opa-config.git, and the branch should match that defined in [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment.git) _apps/opal/envs/$ENV/kustomization.yaml_ patch.

### AWS SES (Simple Email Service) Production Access

Our infrastructure includes a Terraform deployment that sets up the necessary components for sending emails using AWS SES from a subdomain. For example it would be used to send automated emails to users that create new billing accounts on the EO-DataHub platform.

Due to AWS SES defaults, additional action is required before you can send emails to arbitrary recipients. New AWS accounts start with SES in `sandbox` mode, which imposes the following restrictions:
- Emails can only be sent to verified addresses
- Low sending limits
- Useful only for testing, not production use

To enable full email sending capabilities (i.e. to any unverified recipient), you must **request production access** for SES.

#### Move SES Out of Sandbox
1. In the AWS SES Service, click on _Get set up_
2. Click on _Request production access_
3. A form will appear to show the request details
4. Fill out the form with the relevent details and submit the request

AWS typically reviews and approves production access within 24-48 hours but they might ask you to provide additional information to make sure the service is not intended for spamming and other bad practises.
