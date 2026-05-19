### 3.15 System Management Services

A package of supporting components will be installed to provide a number of system management services. Some of these are described elsewhere 

- ArgoCD in 4.3 GitOps Cloud Deployment 
- Linkerd in 3.13.2.8 Intra-service Access 

#### 3.15.1 Autoscaler

The Kubernetes Autoscaler increases the size of EKS node groups when a pod cannot be scheduled, subject to limits configured via Terraform. This means that user workload node groups (the workflow and notebook groups) can be allowed to scale by a large number of nodes to manage costs as user demand changes. The system and TiTiler groups also autoscale but infrequently. 

#### 3.15.2 External Secrets

External Secrets is a tool to link secrets in external secrets stores – AWS Secrets Manager in EODHP’s case – with Kubernetes secrets. Secrets only generated and used within Kubernetes use ordinary Kubernetes secrets. Other secrets are stored in AWS Secrets Manager and copied into Kubernetes Secrets as required. AWS Secrets Manager has a UI so is used for manually adding secrets such as Dockerhub credentials, and it also integrates with other services such as Aurora so that AWS auto-generated credentials can be found there. 

#### 3.15.3 Metric-Based Monitoring

Prometheus is installed in the cluster and scrapes monitoring data from a wide variety of components, including Kubernetes core components and Pulsar. These are then available for querying and charting in Grafana. Initial custom dashboards can be defined in ArgoCD. 

This allows for monitoring of cluster resource use for each namespace, pod, etc. It also allows for monitoring of Pulsar message backlogs and delivery rates. 

The CPU and memory data from the core Kubernetes monitoring are also used for billing, with the accounting system querying Prometheus for them. 

#### 3.15.4 Logging

Pod logs are sent to stdout/stderr and are both harvested by the Elastic Stack and are available through the usual kubectl log commands. The Elastic Stack allows for searching of logs across the whole cluster in a single interface. 

Some components, particularly those in the accounting system and harvest pipeline, produce JSON structured log data through Open Telemetry. This allows more structured querying in Kibana, for example by workspace. OTEL span data is also emitted allowing logs for a single API call to be correlated across multiple microservice hops. This also provides enough data for distributed tracing, although this is not configured into Grafana at present. 

#### 3.15.5 AWS CloudWatch Canaries

AWS CloudWatch provides Canaries, which are lambdas run every 5 minutes to check a service and provide a success/failure indication. Canaries are configured to check that each publicly available service in EODH is able to return a response. In the event of failure, CloudWatch is configured to send an alert to an SNS topic which in turn emails a list of system operators. 

This is configured manually in the AWS UI. 

