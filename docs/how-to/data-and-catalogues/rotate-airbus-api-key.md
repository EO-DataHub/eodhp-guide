---
title: Rotate the Airbus shared API key
doc_status: unreviewed
tags:
  - commercial-data
  - kubernetes
  - maintenance
last_reviewed:
reviewed_by:
review_notes:
---

# Rotate the Airbus shared API key

How to replace the Airbus shared system API key when it expires or is approaching expiry.

**See also:** [Airbus Adaptor](../../reference/services/airbus-adaptor.md) — for the per-workspace OTP key mechanism used for placing orders, which is separate from the shared key covered here.

---

## Background

The Airbus purchasing workflow uses two distinct keys:

| Key | Purpose | Storage |
|-----|---------|---------|
| **Shared system key** | Fetch country list, validate purchase inputs | Kubernetes Secret (`rc` namespace) |
| **Per-workspace key** | Place actual purchase orders | AWS Secrets Manager (OTP-encrypted) |

When the shared key expires, all purchase-related UI requests fail before reaching the order step — causing data ordering errors and missing quicklook previews. Airbus keys cannot be renewed; a new key must be created to replace the expired one.

There is currently no automated expiry monitoring or alerting. Track the key's expiry date manually and schedule rotation before it expires.

---

## Prerequisites

- Access to the [Airbus Developer Portal](https://www.geoapi-airbusds.com/getting-started/authentication/)
- `kubectl` access to the cluster with write permissions on Secrets in the `rc` namespace

---

## Steps

### 1. Create a new key on the Airbus Developer Portal

Log in to the Airbus Developer Portal and generate a new API key. Copy the key value — it is only shown once.

### 2. Identify the current Kubernetes Secret

```bash
kubectl get secrets -n rc | grep airbus
```

The shared key is stored in the `airbus-secrets` secret under the key `airbus-api-key`.

### 3. Update the secret

```bash
kubectl patch secret airbus-secrets -n rc \
  --type='json' \
  -p='[{"op": "replace", "path": "/data/airbus-api-key", "value": "'$(echo -n "<new-key>" | base64)'"}]'
```

Replace `<new-key>` with the plaintext key value from step 1.

### 4. Verify

Confirm the Airbus purchasing flow is working by attempting a country list fetch or a test purchase validation in the RC UI. Check the adaptor pod logs for any remaining authentication errors:

```bash
kubectl logs -n rc -l app=airbus-adaptor --tail=50
```

---

## Notes

- The per-workspace OTP key mechanism (used for placing orders) is managed separately — see the [Airbus Adaptor reference](../../reference/services/airbus-adaptor.md#configuration-kubernetes-secrets) for details.
- Proposed future improvements include expiry tracking in Secrets Manager, a Grafana/Prometheus alert at 30 days before expiry, and automated renewal via the Airbus key management API. These are not yet implemented.
