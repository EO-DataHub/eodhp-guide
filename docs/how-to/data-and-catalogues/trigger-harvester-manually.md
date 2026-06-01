---
title: Manually trigger a harvester
doc_status: unreviewed
tags:
  - stac
  - data-catalogues
  - kubernetes
last_reviewed:
reviewed_by:
review_notes:
---

# Manually trigger a harvester

How to trigger an out-of-schedule harvest run — useful after depositing NPL QA files or when testing catalogue updates without waiting for the next scheduled run.

See also: [NPL QA Assessments as STAC Collection Assets](../../explanation/architecture/catalogue/npl-qa-collection-assets.md) — harvest timing per collection.

## Prerequisites

- `kubectl` access to the cluster
- Permission to create Jobs in the `rc` namespace

---

## CronJob-based harvesters (CEDA, Planet)

CEDA and Planet harvesters run as Kubernetes CronJobs. Create a one-off Job from the existing CronJob:

```bash
# List available CronJobs to confirm the name
kubectl get cronjobs -n rc

# Trigger a run (adjust name and CronJob reference as needed)
kubectl create job planet-manual-1 --from=cronjob/planet-harvester -n rc
```

Give the Job a unique name each time — Kubernetes will reject a duplicate name.

---

## Argo Events-based harvesters (Airbus)

Airbus harvesters are triggered via Argo Events and cannot be re-triggered from a CronJob. Instead, create a Job directly with the harvester image.

Set `HARVESTER_CONFIG_KEY` to the product you want to harvest:

| Product | `HARVESTER_CONFIG_KEY` |
|---------|----------------------|
| Pleiades HR | `PHR` |
| Pleiades Neo | `PNEO` |
| SPOT | `SPOT` |
| SAR | `SAR` |

Give the Job a unique `metadata.name` each run.

```bash
kubectl create -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: airbus-phr-manual-01   # change for each run
  namespace: rc
spec:
  ttlSecondsAfterFinished: 86400
  template:
    spec:
      serviceAccountName: resource-catalogue
      automountServiceAccountToken: true
      restartPolicy: Never
      containers:
        - name: airbus-harvester
          image: public.ecr.aws/eodh/airbus-harvester:latest
          imagePullPolicy: Always
          securityContext:
            runAsUser: 50004
            runAsGroup: 50004
          args:
            - default_workspace
            - catalog
            - catalogue-population-eodhp
          resources:
            requests:
              cpu: 100m
              memory: 256Mi
            limits:
              memory: 4Gi
          env:
            - name: PULSAR_URL
              value: pulsar://pulsar-broker.pulsar:6650
            - name: MINIMUM_MESSAGE_ENTRIES
              value: "100"
            - name: HARVESTER_CONFIG_KEY
              value: "PHR"   # change to PNEO, SPOT, or SAR as needed
            - name: PROXY_BASE_URL
              value: https://eodatahub.org.uk
            - name: COMMERCIAL_CATALOGUE_ROOT
              value: "catalogs/commercial"
            - name: AIRBUS_API_KEY
              valueFrom:
                secretKeyRef:
                  name: airbus-secrets
                  key: airbus-api-key
            - name: TOPIC
              value: "bulk"
  backoffLimit: 4
EOF
```

---

## Checking the result

After the Job completes, confirm the collection was updated in the STAC catalogue. For QA asset updates, check that the `qa_documentation` or `qa_radiometric` assets now appear on the relevant collection record.
