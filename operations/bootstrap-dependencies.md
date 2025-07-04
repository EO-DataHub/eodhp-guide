# Bootstrap Dependencies

Dependencies between some services need to be managed carefully so that clusters can be bootstrapped. This describes some of those dependencies and the order we require. These are mostly configured into ArgoCD and do not need to be explicitly handled by a system operator.

If you need to debug a badly broken platform and you don't know why then this is a useful order to check things in - later items will not be the cause of problems in earlier items.

See [Platform Deployment](Platform Deployment.md) for a step-by-step cluster creation guide.

Some particular concerns:

- Nothing that is installed before linkerd should depend on linkerd or its admission webhook.
  Annotating it `linkerd.io/inject: disabled` is not sufficient - the webhook will still be called
  for any new or change to a Pod or Service which isn't specifically excluded in the webhook's
  config. This exclusion is configured in eodhp-argocd-deployment/apps/linkerd/base/control-plane/values.yaml,
  usually by namespace.
- The cluster autoscaler won't work until the `autoscaler` namespace is working. Worst case is that
  this can't start because all the nodes are full, so better to start it early.
- Many certs are created in the `certs` namespace and copied by `replicator`. Replicator must be
  able to start without anything which needs any of these certificates.
- cert-manager needs the AWS IAM integration running so it can talk to Route 53.

ArgoCD has 'sync waves' where earlier sync waves must result in healthy K8s resources before later
ones will happen.

Bootstrap order:

- The AWS account must be created.
- The Terraform supporting infrastructure config is applied.
- AWS secrets and domain names are manually configured.
- The per-cluster Terraform installs some Kubernetes components:
  - aws-ebs-csi-driver and two EBS storage classes
  - aws-efs-csi-driver and its storage class
  - the nginx namespace and the LoadBalancer service linked to ELB - note that this is initially
    created without the linkerd annotation.
- ArgoCD is installed manually first. The web interface will not work but it will start
  synchronizing from Git. `kubectl port-forward svc/argocd-server -n argocd 8080:443` may be
  useful to get the web UI and use its CLI.
- sync-wave -100 (k8s-level infra CRDs):
  - AWS ACK (Amazon Controllers for Kubernetes) CRDs.
  - cert-manager and trust-manager CRDs.
  - linkerd CRDs.
- sync-wave -95 (k8s-level infra AWS IAM roles, needs the IAM CRDs):
  - AWS ACK IAM controller
  - IAM Roles needed by the other ACK controllers.
- sync-wave -90 (k8s-level infra controllers that need the IAM roles):
  - Other AWS ACK controllers (EFS, EKS, and S3).
  - AWS Autoscaler.
  - cert-manager and trust-manager.
  - Certificates in the `certs` and `linkerd` apps.
  - replicator
  - secret-generator
- sync-wave -50 (linkerd):
  - linkerd, but not linkerd-viz
- sync-wave -40 (management infra datastores):
  - external-secrets
  - `database` app (creates DBs in Aurora)
  - redis (needed by oauth2-proxy)
- sync-wave -30 (management infra):
  - nginx
  - ArgoCD
  - Keycloak (note: depends on a database and external-secrets)
  - ELK
  - Prometheus
  - Grafana (needs external-secrets)
  - linkerd viz
- sync-wave -20 (EODH infra)
  - OPAL
  - Pulsar
  - oauth2-proxy
- sync-wave 0 (default): everything else
- If required for the cluster, AWS Synthetics Canaries monitoring must be manually configured.
