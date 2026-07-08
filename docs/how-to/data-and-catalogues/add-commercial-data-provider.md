---
title: Add a new commercial data provider
doc_status: unreviewed
last_reviewed:
reviewed_by:
review_notes:
tags:
  - commercial-data
  - workflows
---
# Add a new commercial data provider

Checklist for onboarding a new commercial data provider adaptor (e.g. a fourth provider alongside Airbus, Planet, Open Cosmos) so its purchase workflow runs with the correct permissions and status tracking from the first deployment.

**See also:** [Execution model: public vs user-service](../../explanation/architecture/commercial-data-purchasing.md#execution-model-public-vs-user-service) · [Workflow Runner (ADES) reference](../../reference/services/workflow-runner.md) · [Data Adaptors reference](../../reference/services/data-adaptors.md)

## Steps

1. **Create the provider workspace** (e.g. `open-cosmos`) if it doesn't already exist. Deploying the CWL requires a workspace-scoped token for the owning workspace.

2. **Wire the provider into `resource-catalogue-fastapi`** The Purchase API hardcodes providers in several places (`resource_catalogue_fastapi/models.py` and `__init__.py`), all of which need a new arm for the provider before an order can be placed at all:
     - Add the provider to the `OrderableCatalogue` enum, and add an `Orderable<Provider>Collection` enum (or equivalent) listing which of its collections are orderable. This list is maintained by hand, not populated from the catalogue (`# TODO: This should be populated from the catalogue`) — adding a new orderable collection later, even for an existing provider, needs a code change and release, not just a catalogue update.
     - Add the provider's arm to the per-provider `if`/`elif catalog.value == OrderableCatalogue.X.value` branches in `quote()` and `order_item()` — pricing/quote logic and adaptor selection are both branched per provider. `collection` is typed as the combined `OrderableCollection` enum, so an unregistered collection gets rejected with a 422 before any provider code runs.
     - **The quote endpoint calls the provider's pricing API synchronously, from inside the request handler** — it can't go through the CWL workflow the way ordering does. Write the provider's quote/pricing call as its own small `<provider>_client.py` module in this repo (`open_cosmos_client.py` is the template: a single `<provider>_get_quote(workspace, collection_id, item_id) -> QuoteResponse` function).
     - **If the provider doesn't use the Airbus/Planet OTP-encrypted static-API-key model**, the standard `get_api_key()` checks in `quote()` and `order_item()` need an explicit bypass for it (see how Open Cosmos, which uses OAuth2 client-credentials + refresh tokens instead, is excluded via `if catalog.value != OrderableCatalogue.open_cosmos.value`).
     - **A provider with refreshable credentials needs the read/refresh logic written twice, not once.** The quote path above runs inside `resource-catalogue-fastapi` and can't call into the CWL adaptor, so it needs its own copy of the credential read/refresh code — separate from the adaptor's `auth_utils.py` (step 3). For Open Cosmos, `resource_catalogue_fastapi/open_cosmos_client.py` and `open-cosmos/open_cosmos_adaptor/auth_utils.py` are two independent implementations that both read and refresh the same `oauth-<provider>` Kubernetes Secret — a fix to one doesn't cover the other. Refreshing persists the new token back to that Secret; the `V1Secret` object passed to the Kubernetes replace call must include `metadata.name`/`metadata.namespace`, or the API rejects it.
     - Check whether the provider's ordering model is 1:1 with a catalogue item. Airbus/Planet append a tagged, hashed item ID to support repeat purchases of the same item with different options — Open Cosmos orders don't support this and the tag has to be explicitly suppressed for that provider.

3. **Write the adaptor code.** Each provider's adaptor is an independent Python package under its own top-level folder (e.g. `open-cosmos/`, `planet/`, `airbus/`) — there's no shared library across providers to build on (Airbus's two adaptors share `airbus/common/`, but Open Cosmos and Planet each maintain their own separate `auth_utils.py`/`s3_utils.py`/`stac_utils.py`). Use an existing adaptor as a template — Open Cosmos's package is the simplest and most recently written. At minimum you need:
     - `__main__.py` — orchestrates one order per STAC item found in the input catalogue: submit → wait for fulfilment → download → upload → update STAC status at each stage, catching and recording a failure per item rather than aborting the whole batch.
     - `auth_utils.py` — provider API authentication (API key exchange, OAuth2 client-credentials, or whatever the provider requires), reading secrets from environment variables sourced from Kubernetes Secrets.
     - `s3_utils.py` — upload/download against the workspace and commercial-data S3 buckets.
     - `stac_utils.py` — updates to the STAC Order extension fields (`order:status`, `order:id`, `order:date`) and asset records at each pipeline stage, writing to S3 and sending the Pulsar `transformed` notification.
     - `Dockerfile` + `requirements.txt` — mirror an existing adaptor's Dockerfile (`python:3.12-slim`, copy `requirements.txt`, `pip install -r requirements.txt`, copy source, `ENTRYPOINT ["python", "-m", "<package>"]`).
     - A `README.md` in the adaptor's folder describing its execution model and inputs, plus an entry in the repo's root `README.md` adaptor table.

4. **Match the CWL's `inputBinding` to the adaptor's CLI.** The Python entrypoint expects named flags (e.g. `--coordinates`, `--catalogue_dirs`), not bare positional args — mirror an existing adaptor's CWL (e.g. `airbus-optical.cwl`) for the `prefix:` fields. If the CLI's argparse declares fewer positional arguments than the CWL passes, later arguments get silently corrupted rather than raising an error — check the argument counts match. When trimming inputs the CLI doesn't take as an argument, check whether they're still needed as an *environment variable* injection first (e.g. `cluster_prefix` is consumed via the injected `CLUSTER_PREFIX` env var, not a CLI flag).

5. **Build and publish the Docker image — this isn't automated.** The repo's CI (`.github/workflows/actions.yaml`) only runs QA and security-scan checks; nothing builds or pushes adaptor images. Build and tag the image manually and push it to `public.ecr.aws/eodh/<adaptor-name>` (see the root `README.md` for the `docker build`/`tag`/`push` commands), then update both the CWL's `dockerPull` tag and its `s:softwareVersion` before deploying.

6. **Deploy the CWL**: `POST {WORKFLOW_RUNNER_API_URL}/{workspace}/processes`, using the workspace-scoped token from step 1.

7. **Deploy it as a user-service, not public.** Upload an access-policy.json with `"user_service": true, "public": false` and `"id": "<workflow-id>"` to `s3://user-workflows-catalogue-<cluster_prefix>/deployed/<workspace>/<workflow_id>.access-policy.json`. Without this, the workflow silently falls back to running in the *calling user's* workspace instead of the provider's — it won't fail loudly, it just won't have the provider secrets it needs. See [Execution model: public vs user-service](../../explanation/architecture/commercial-data-purchasing.md#execution-model-public-vs-user-service) for why this matters, and the [access-policy schema reference](../../reference/services/workflow-runner.md#access-policies-public-vs-user-service) for the full mechanism. This can be done manually (`aws s3 cp` / boto3) or via the `public-workflow-loader` CLI, which deploys the CWL and uploads the access-policy together in one step.

8. **Register the provider in the output-path allowlist.** `eoepca-proc-service-template`'s `service.py` hardcodes which provider workspaces get `commercial-data` as their stage-out S3 prefix — anything not on the list falls back to `processing-results/<workflow_id>`, which leaves the order-tracking STAC item's `order:status` stuck on `pending` forever even though the workflow itself succeeds. Add the new provider's workspace name to that allowlist. See [Workflow Runner reference — known gotchas](../../reference/services/workflow-runner.md#known-gotchas) for the allowlist location and a template-caching bug that can make this change appear not to take effect after redeploying.

9. **Test end-to-end** — see [Test commercial data adaptor changes](test-commercial-data-adaptor.md). Test both the quote and order endpoints; each has its own per-provider branch in `resource-catalogue-fastapi` (step 2), so a working order doesn't confirm the quote path also handles the new provider. Confirm both that the workflow succeeds *and* that the purchase itself completes (a successful workflow run doesn't guarantee a successful purchase), and confirm `order:status` progresses past `pending` with assets landing under the `commercial-data/` prefix.
