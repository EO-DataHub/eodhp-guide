# Workflow Runner (ADES)

## Summary

The workflow runner is built on the EOEPCA ADES building block. It orchestrates workflow execution for the platform. A REST API has been built on top of the ADES component for managing workflow requests and integrating it with the platform IAM.

### Dependent Services

- Workflow executions will not work if the workflow runner is not operational
- Harvesting pipelines, such as adapters, rely on the Workflow Runner to run

## Operation

The Workflow Runner is comprised of several services, which all run in the `ades` namespace of the Kubernetes cluster.

- deployment.apps/zoo-project-dru-zookernel
- deployment.apps/zoo-project-dru-zoofpm
- deployment.apps/zoo-project-dru-kubeproxy
- deployment.apps/workflow-runner-api
- deployment.apps/workflow-ingester

The `workflow-runner-api` exposes a REST API that receives requests and processes them. Workflows can then be deployed, discovered and executed via the API, using HTTPS requests, Jupyter Notebooks, or via the Python pyeodh client.

To bugfix a failed workflow in the backend, you will need to know the workspace, `<workspace-name>`, in which the workflow was executed which can be used to determine the namespace in which the workflow was run, `ws-<workspace-name>`. You can then list all of the pods that were recently run in that workspace to identify any useful pods, `kubectl -n ws-<workspace-name> get pods`. Look for any recent pods with a naming structure similar to `job-<uuid>`. You can then view the logs for any of these pods using `kubectl -n ws-<workspace-name> logs job-<uuid>` and view logs for each of the steps that were run during the execution of that workflow, this includes Kubernetes manifests for each job run for that workflow as well. You can also view these logs in realtime by providing the `-f` argument. In these logs look for an `Error` alerts or `OOMKilled` warnings to determine what caused a workflow step to fail. If you see any Out of Memory errors, you may need to increase the amount of RAM and CPU being requested by the workflow in the CWL resource requirements - you can check these values also in the printed manifests:

```
resources:
  limits:
    cpu: "1"
    memory: 1Gi
  requests:
    cpu: "1"
    memory: 1Gi
```

You are also able to view workflow logs within the `zoo-project-dru-zoofpm` pod as well. First exec into the pod using `kubectl -n ades exec -it deploy/zoo-project-dru-zoofpm -- bash` and navigate to the `/opt/zooservices_user/<workspace-name>` directory to view all the available workflows in that workspace. You can then navigate to the `temp` directory to view details for any of the jobs that were run within that workspace, you can view the overall job details using the files in this directory, for example `<workflow-id>_<job-id>_error.log`. You can also view the individual step logs by navigating to the job-specific directories, for example `/opt/zooservices_user/<workspace-name>/temp/<workflow-id>-<job-id>`.

### Configuration

The Auth Agent is configured as part of the [ArgoCD deployment repo](https://github.com/EO-DataHub/eodhp-argocd-deployment) in the apps/ades directory.

### Control

To restart a Workflow Runner service run `kubectl rollout restart -n ades <service-fqdn>` for Kubernetes cluster or use ArgoCD UI to restart, where `service-fqdn` is one of the following, relating to the service to be restarted:

- deployment.apps/zoo-project-dru-zookernel
- deployment.apps/zoo-project-dru-zoofpm
- deployment.apps/zoo-project-dru-kubeproxy
- deployment.apps/workflow-runner-api
- deployment.apps/workflow-ingester

To stop service, the service must be removed from ArgoCD configuration.

### Dependencies

- Open Policy Agent - The Workflow Runner uses fine-grained authorisation policies defined in the Open Policy Agent. If the OPA is not available, authorisation of requests will fail.
- S3 access to workflow access bucket - Access to historic workflow logs and results is determined based on workflow access details stored in S3

### Backups

All state for the Workflow Runner is in its database. Restoring a previous state involves following the database restore procedure.

## Development

### Code Repositories and Artifacts

The Workflow Runner is built from multiple microservices, each with their own code repository.

- ZOO-Project

  - The ZOO parent git repository - https://github.com/EO-DataHub/ZOO-Project
    - Image: public.ecr.aws/eodh/zoo-project-dru
  - Pycalrissian - https://github.com/EO-DataHub/pycalrissian
  - ZOO Calrissian Runner - https://github.com/EO-DataHub/zoo-calrissian-runner
  - Calrissian - https://github.com/EO-DataHub/calrissian
    - Image: public.ecr.aws/eodh/eodhp-calrissian
  - Proc Service Template - https://github.com/EO-DataHub/eoepca-proc-service-template

- Stage In and out

  - Stagein - https://github.com/EO-DataHub/ades-stagein
    - Image: public.ecr.aws/eodh/eodhp-ades-stagein
  - Stageout - https://github.com/EO-DataHub/ades-stageout
    - Image: public.ecr.aws/eodh/eodhp-ades-stageout

- Workflow Runner API - https://github.com/EO-DataHub/eodhp-argocd-deployment/tree/main/apps/ades/base/api

  - Deployment
    - Image: public.ecr.aws/eodh/ades-fastapi
  - Service
  - Ingress

- System Tests - https://github.com/EO-DataHub/eodhp-argocd-deployment/tree/staging/apps/ades/base/tests

  - Workflow System Test - https://github.com/EO-DataHub/workflow-system-test
    - Image: public.ecr.aws/eodh/workflow-system-test

- Workflow Harvester - https://github.com/EO-DataHub/eodhp-argocd-deployment/tree/staging/apps/ades/base/harvester

  - Git Change Scanner
    - Image: public.ecr.aws/eodh/eodhp-git-change-scanner

- Workflow Ingester
  - Deployment
    - Image: public.ecr.aws/eodh/eodhp-workflow-ingester

Additional guidance can be found in the [EOEPCA Deployment Guide](https://eoepca.readthedocs.io/projects/deploy/en/stable/eoepca/ades-zoo/) under the ADES component.
