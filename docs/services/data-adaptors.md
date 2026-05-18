# Data Adaptors

## Summary

Data adaptors enable ordering of commercial data from Airbus and Planet within the EO DataHub. These adaptors run as user service workflows in specialised data provider workspaces via the [Workflow Runner](workflow-runner.md) service.

Each adaptor interfaces with its respective provider to place an order for a single item, waits for delivery to an S3 bucket, then downloads and attaches assets to a STAC item that tracks the order. The Workflow Runner manages asset upload and ingestion of STAC items into the user's workspace.

### Code Repositories and Artifacts

- Source code: [EO-DataHub/commercial-data-adaptors](https://github.com/EO-DataHub/commercial-data-adaptors)

**Airbus:**
- Container images:
  - `public.ecr.aws/eodh/airbus-optical-adaptor`
  - `public.ecr.aws/eodh/airbus-sar-adaptor`

**Planet:**
- Container image:
  - `public.ecr.aws/eodh/planet-adaptor`

### Dependent Services

There are no services that depend on the adaptors.

## Operation

Adaptors run as workflows under either the `ws-planet` or `ws-airbus` namespaces. Once deployed, they can be started by making calls to an endpoint in the `manage-catalogue-fastapi` API.

### Configuration

Adaptors are configured as user service workflows using HTTP and CWL scripts provided in the adaptor repository.

### Control

Adaptors are managed and executed by the Workflow Runner service and the `manage-catalogue-fastapi` API in the Resource Catalogue namespace.

### Dependencies

- **Workflow Runner:** Adaptors run as workflows within the EODH.
- **STAC FastAPI:** Outputs from the adaptors are ingested into the ordering workspace's catalogue.
- **Resource Catalogue (`manage-catalogue-fastapi`):** Adaptors are triggered by this API, which handles inputs and the creation of the order-tracking STAC item.
- **Workspace Controller:** Adaptors require data provider API keys linked to a workspace. Orders cannot be placed if a key is not linked.

### Airbus workspace secrets (operator reference)

When a workspace user adds Airbus credentials in the UI, the platform stores them using a split between Kubernetes and AWS Secrets Manager. The adaptor implementation is in [`airbus/common/auth_utils.py`](https://github.com/EO-DataHub/commercial-data-adaptors/blob/main/airbus/common/auth_utils.py) in the commercial-data-adaptors repository.

**Namespace**

For a workspace named (for example) `airbus-order-testing`, adaptors and related resources use the Kubernetes namespace `ws-airbus-order-testing` (prefix `ws-` plus the workspace name).

**Kubernetes secret `otp-airbus`**

In namespace `ws-{workspace}`:

| Key | Purpose |
| --- | --- |
| `otp` | Base64 one-time pad. XOR with the ciphertext from AWS recovers the **plaintext Airbus API key**. |
| `contracts` | Base64-encoded JSON used as Airbus **contract configuration** (the optical adaptor reads e.g. the `"optical"` section). |

The cleartext API key is not present in either store alone: K8s holds the pad and contracts; AWS holds ciphertext keyed by provider name.

**AWS Secrets Manager**

- **Secret id:** `ws-{workspace}-{CLUSTER_PREFIX}` (typical suffix `eodhp` when `CLUSTER_PREFIX` is unset or left at the adaptor default; the adaptors’ `CLUSTER_PREFIX` environment variable overrides the suffix).
- **SecretString:** JSON. Commercial adaptors expect a property **`airbus`**: base64 ciphertext that pairs with the Kubernetes `otp` value (same decoded byte lengths) via XOR.

Under a normal split deployment, recovering a user’s API key effectively requires access to **both** the Kubernetes secret (`otp-airbus` / `otp`) **and** the Secrets Manager secret (`airbus` ciphertext).

**Adaptors**

| Adaptor | `contracts` | Auth |
| --- | --- | --- |
| Optical | Yes (`get_airbus_contracts`) | `generate_access_token` using API key from OTP + AWS (`get_airbus_api_key`) |
| SAR | Not read in SAR `api_utils` | Same token path via API key |

Token acquisition uses Airbus OneAtlas authenticate endpoints; `env` selects prod vs non-prod URLs.

**Sanity checks**

```bash
kubectl -n ws-<workspace-name> get secret otp-airbus -o yaml
# expect data keys: contracts, otp
```

In AWS, locate secret `ws-<workspace-name>-eodhp` (or `ws-<workspace-name>-<CLUSTER_PREFIX>`) and confirm JSON includes key `airbus`.

### Backups

Adaptor outputs are stored in S3 buckets. Ingested items are backed up as part of the `stac-fastapi` database. The original delivered items are not removed by the adaptors.

## Development

Adaptor code is version controlled in the [EO-DataHub/commercial-data-adaptors](https://github.com/EO-DataHub/commercial-data-adaptors) repository.

New versions are released by following the release process described in the repository's README. Deploying adaptors is a manual one time process that must be done when all dependencies are deployed and workspaces for data providers are created.

In order to deploy adaptors, `planet` and `airbus` workspaces must first exist. It is useful but not necessary to make a specific user with these names to own each of these workspaces to maintain security and allow admins to deploy the adaptors with a workspace scoped token. Login credentials for these accounts can be managed via keycloak.
