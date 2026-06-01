---
title: Workspace Cost API
doc_status: unreviewed
tags:
  - workspaces
  - kubernetes
  - observability
last_reviewed:
reviewed_by:
review_notes: Service not yet deployed to production.
---

# Workspace Cost API

Explains the workspace cost API (`kubecost-eodh-api`) — a service that gives workspace members scoped visibility into the Kubernetes resource costs for their workspaces, backed by Kubecost.

**Status:** In development. Kubecost is installed in the cluster; this API is not yet live.

**Reference:** [Workspaces service](../../reference/services/workspaces.md)

---

## Purpose

Kubecost collects per-namespace cost allocation data across the cluster. The workspace cost API sits in front of Kubecost and enforces that callers only see cost data for namespaces belonging to workspaces they are members of. It presents a stable, workspace-oriented response shape rather than exposing raw Kubecost output.

The intended consumers are workspace members who want to understand what their notebooks, workflows, and storage are costing — without needing direct Kubecost access.

---

## How authentication and scoping work

All cost endpoints require a Keycloak Bearer token:

```
Authorization: Bearer <JWT>
```

The service verifies the token signature using RS256 against the Keycloak JWKS endpoint, then derives an **authorization context** from the JWT claims:

| Claim | Used for |
|-------|---------|
| `workspace` / `workspaces` | The set of workspace IDs the caller is allowed to see |
| `namespaces` | Override list of allowed namespaces (optional; computed as `ws-<workspaceId>` if absent) |
| `token_type` | Must be `user` or `machine`; anything else is rejected with 403 |

When a request arrives for namespace `ws-abc`, the service checks that `abc` is in the caller's allowed workspaces and that `ws-abc` is in their allowed namespaces before forwarding to Kubecost. Requests for namespaces outside the caller's token scope return 403.

---

## API endpoints

Base path: `/v1/costs`

### All workspace costs

```
GET /v1/costs/workspaces?start=<ISO8601>&end=<ISO8601>
```

Returns the total cost for every workspace the caller's token is permitted to see, in a single Kubecost call. Workspaces with no data in the window return a zero entry rather than being omitted.

**Response:**
```json
{
  "window": { "start": "2026-05-01T00:00:00Z", "end": "2026-06-01T00:00:00Z" },
  "workspaces": [
    { "workspaceId": "my-workspace", "totalCost": 48.32 },
    { "workspaceId": "another-ws",  "totalCost": 12.10 }
  ],
  "totalCost": 60.42
}
```

### Single namespace cost

```
GET /v1/costs/namespace/{namespace}?start=<ISO8601>&end=<ISO8601>
```

Returns a detailed cost breakdown for one namespace. The namespace must follow the `ws-<workspaceId>` pattern and the caller must be a member of that workspace.

**Response:**
```json
{
  "workspaceId": "my-workspace",
  "namespace": "ws-my-workspace",
  "window": { "start": "2026-05-01T00:00:00Z", "end": "2026-06-01T00:00:00Z" },
  "totalCost": 48.32,
  "breakdown": {
    "cpuCost": 20.10,
    "ramCost": 15.40,
    "pvCost": 12.00,
    "networkCost": 0.82,
    "loadBalancerCost": 0.00,
    "externalCost": 0.00,
    "adjustments": { "cpuCostAdjustment": 0.0, "ramCostAdjustment": 0.0, "pvCostAdjustment": 0.0 }
  },
  "topDrivers": [
    { "name": "CPU", "cost": 20.10, "share": 0.416 },
    { "name": "Memory", "cost": 15.40, "share": 0.319 },
    { "name": "Persistent Volumes", "cost": 12.00, "share": 0.248 }
  ],
  "notes": ["CPU is the top cost driver."]
}
```

The `start`/`end` query parameters accept ISO 8601 datetimes. Alternatively, a combined `window=start,end` parameter can be used. The maximum window span is 92 days.

### Health and readiness

```
GET /health   # liveness — always 200 if the process is running
GET /ready    # readiness — 503 if Kubecost is unreachable
```

---

## How Kubecost is queried

The service calls Kubecost's `/model/allocation` endpoint with:

- `aggregate=namespace` — groups cost by Kubernetes namespace
- `accumulate=true` — sums costs over the window rather than returning per-interval buckets
- `idle=false` — excludes idle cluster cost
- `filter=namespace:"ws-abc"` (or a comma-separated list for the workspaces endpoint)

Costs are broken down into CPU, memory, persistent volumes, network, load balancer, and external costs, with per-component adjustment values. The `topDrivers` list in the single-namespace response is computed from this breakdown and sorted by cost share.

Kubecost calls are retried up to twice with exponential backoff. If Kubecost is unreachable after retries, the API returns 502.

---

## Configuration

Key environment variables:

| Variable | Purpose | Default |
|----------|---------|---------|
| `KUBECOST_BASE_URL` | In-cluster Kubecost URL | `http://127.0.0.1:9090` |
| `JWT_JWKS_URI` | Keycloak JWKS endpoint | (required) |
| `OIDC_DISCOVERY_URL` | OIDC discovery URL (alternative to `JWT_JWKS_URI`) | — |
| `JWT_ISSUER` | Expected `iss` claim | (optional) |
| `JWT_AUDIENCE` | Expected `aud` claim | (optional) |
| `JWT_JWKS_CACHE_TTL_SECONDS` | JWKS key cache lifetime | `300` |
| `KUBECOST_TIMEOUT_SECONDS` | Per-request timeout | `15.0` |
| `KUBECOST_RETRIES` | Retry attempts on failure | `2` |
| `MAX_WINDOW_DAYS` | Maximum query window | `92` |
| `LOG_LEVEL` | Log verbosity | `INFO` |

For production, `KUBECOST_BASE_URL` should point to the in-cluster Kubecost service: `http://kubecost-cost-analyzer.kubecost.svc.cluster.local:9090`.
