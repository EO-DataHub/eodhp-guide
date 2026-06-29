---
title: Access Platform Logs
doc_status: ok
tags:
  - victorialogs
  - grafana
  - observability
last_reviewed: 2026-06-26
reviewed_by: recmanj
review_notes: Replaces access-kibana-logs.md; updated for VictoriaLogs/Grafana stack.
---
# Access Platform Logs

## Purpose

Platform logs from all Kubernetes pods are collected by Vector, stored in VictoriaLogs, and surfaced through a Grafana dashboard. This procedure explains how to open the log explorer.

## When to Use

Use this when you need to view or search platform logs — container output, application errors, or audit trails across services.

## Requirements

- A Hub account with the `admin` realm role in Keycloak (see [Elevate user](../identity-access/elevate-user.md) if an operator must grant it)

## Operation

1. Visit https://grafana.eodatahub.org.uk/ and sign in with your Hub admin account via Keycloak.
2. In the left-hand menu, go to **Dashboards**.
3. Open the **VictoriaLogs** folder.
4. Select **EODH Platform Logs Explorer**.

The dashboard opens with all platform logs visible. Use the dropdowns at the top to narrow by **Namespace**, **Pod**, **Container**, **Level**, and **Stream**, or type a [LogsQL](https://docs.victoriametrics.com/victorialogs/logsql/) expression in the **Search** box.

## Useful Information

- Log data is retained for **7 days**.
- Authentication uses your Hub Keycloak account — no separate credentials are required.
- The VictoriaLogs datasource is also available in other Grafana dashboards and panels if you want to build custom views (see [Add a custom dashboard](add-a-custom-dashboard.md)).

**Related:** [Search and filter logs](discover-logs.md).
