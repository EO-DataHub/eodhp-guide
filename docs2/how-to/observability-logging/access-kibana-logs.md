---
title: Access Kibana Logs
doc_status: remove-candidate
tags:
  - elk
  - observability
last_reviewed:
reviewed_by:
review_notes: Migrated from docs/operations/observability/access-kibana-logs.md (legacy doc_status retained).
---
# Access Kibana Logs

## Purpose

The ELK stack has been installed on the platform to provide observability of platform logs and metrics. Kibana (part of the ELK stack) can visualise platform logs and give insight into behaviour. This procedure explains how to open the Kibana UI. It does not cover configuring dashboards — see Kibana’s own documentation for that.

## When to Use

Use this procedure when you need access to the Kibana UI deployed as part of the platform.

## Operation

1. For initial access to Kibana, retrieve the default user credentials from the Kubernetes cluster. The default username is `elastic`. Obtain the password with:

   ```sh
   kubectl -n elk get secret elasticsearch-es-elastic-user -o jsonpath="{.data.elastic}" | base64 -d; echo
   ```

2. Open https://logs.eodatahub.org.uk and sign in with those credentials.
3. Use **Analytics** and **Observability** apps to view logs and metrics for the platform.
4. You can configure dashboards for the behaviour you care about monitoring at a glance.

## Requirements

- Access to the Kubernetes cluster via kubectl

## Useful Information

- The ECK operator requires an enterprise licence for OIDC integration. Use the pre-configured authentication flows here rather than Hub platform OIDC.
- Building useful dashboards and index views is highly use-case specific; follow [Kibana documentation](https://www.elastic.co/guide/en/kibana/current/index.html) for detail.

**Related:** [Discover logs](discover-logs.md).
