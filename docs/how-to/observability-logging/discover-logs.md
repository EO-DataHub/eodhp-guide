---
title: Search and Filter Logs
doc_status: ok
tags:
  - victorialogs
  - grafana
  - observability
last_reviewed: 2026-06-26
reviewed_by:
review_notes: Rewritten for VictoriaLogs/Grafana; replaces Kibana Discover workflow.
---
# Search and Filter Logs

## Purpose

This guide shows how to search and filter platform logs in the Grafana **EODH Platform Logs Explorer** dashboard, using namespace/pod/container dropdowns and LogsQL queries.

## When to Use

Use this when you want to isolate logs from specific services, namespaces, or pods — for example, tracing an error across the authentication layer or inspecting a workflow container's output.

## Requirements

- Access to the EODH Platform Logs Explorer (see [Access platform logs](access-logs.md))

## Operation

### Step 1: Open the dashboard

Follow [Access platform logs](access-logs.md) to open **EODH Platform Logs Explorer** in Grafana.

### Step 2: Narrow by namespace, pod, and container

The top bar provides dropdown filters applied to all panels:

| Dropdown | Purpose |
|----------|---------|
| **Namespace** | Filter to one or more Kubernetes namespaces |
| **Pod** | Filter to specific pods within the selected namespace(s) |
| **Container** | Filter to a specific container within the selected pod(s) |
| **Level** | Filter by log severity: `fatal`, `error`, `warn`, `info`, `debug`, `unknown` |
| **Stream** | `stdout` or `stderr` |
| **Limit** | Maximum number of log lines returned in the Raw Logs panel |

### Step 3: Search with LogsQL

The **Search** box accepts [LogsQL](https://docs.victoriametrics.com/victorialogs/logsql/) expressions. Examples:

| Query | Matches |
|-------|---------|
| `error` | Any log line containing the word `error` |
| `"connection refused"` | Exact phrase |
| `_msg:~"timeout.*"` | Lines where the message matches the regex `timeout.*` |
| `level:error` | Lines where Vector extracted `level` = `error` |

Combine the Search box with the dropdowns — for example, select namespace `auth` and search `"invalid token"` to see auth errors only.

### Step 4: Read the panels

- **Log Volume by Level / Stream / Namespace / Container** — time-series charts showing log rate, useful for spotting anomalies.
- **Raw Logs** — the full log lines matching all active filters.

## Useful Information

- Pod names for Notebook servers are prefixed `jupyter-`; workflow step pods are prefixed with the step name (e.g. `water-quality-`).
- Vector extracts `level` from structured JSON logs (fields `level`, `severity`, `levelname`) and falls back to regex on the raw message. Unrecognised levels appear as `unknown`.
- To check pod details when you don't know the namespace: `kubectl get pods -A | grep <pod-name>`.

**Related:** [Access platform logs](access-logs.md).
