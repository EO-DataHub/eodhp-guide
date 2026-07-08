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

2. **Match the CWL's `inputBinding` to the adaptor's CLI.** The Python entrypoint expects named flags (e.g. `--coordinates`, `--catalogue_dirs`), not bare positional args — mirror an existing adaptor's CWL (e.g. `airbus-optical.cwl`) for the `prefix:` fields. If the CLI's argparse declares fewer positional arguments than the CWL passes, later arguments get silently corrupted rather than raising an error — check the argument counts match.

3. **Deploy the CWL**: `POST {WORKFLOW_RUNNER_API_URL}/{workspace}/processes`, using the workspace-scoped token from step 1.

4. **Deploy it as a user-service, not public.** Upload an access-policy.json with `"user_service": true, "public": false` and `"id": "<workflow-id>"` to `s3://user-workflows-catalogue-<cluster_prefix>/deployed/<workspace>/<workflow_id>.access-policy.json`. Without this, the workflow silently falls back to running in the *calling user's* workspace instead of the provider's — it won't fail loudly, it just won't have the provider secrets it needs. See [Execution model: public vs user-service](../../explanation/architecture/commercial-data-purchasing.md#execution-model-public-vs-user-service) for why this matters, and the [access-policy schema reference](../../reference/services/workflow-runner.md#access-policies-public-vs-user-service) for the full mechanism. This can be done manually (`aws s3 cp` / boto3) or via the `public-workflow-loader` CLI, which deploys the CWL and uploads the access-policy together in one step.

5. **Register the provider in the output-path allowlist.** `eoepca-proc-service-template`'s `service.py` hardcodes which provider workspaces get `commercial-data` as their stage-out S3 prefix — anything not on the list falls back to `processing-results/<workflow_id>`, which leaves the order-tracking STAC item's `order:status` stuck on `pending` forever even though the workflow itself succeeds. Add the new provider's workspace name to that allowlist. See [Workflow Runner reference — known gotchas](../../reference/services/workflow-runner.md#known-gotchas) for the allowlist location and a template-caching bug that can make this change appear not to take effect after redeploying.

6. **Check for a pre-existing IAM role before relying on an ACK-managed one.** If the workspace-controller already auto-created an IAM role for the workspace before a corresponding ACK IAM role resource exists for it in `eodhp-argocd-deployment`, ACK will refuse to manage the role (marks it `Terminal`, requires explicit resource adoption). Confirm whether adoption is needed rather than assuming a newly added ACK resource will apply cleanly.

7. **Don't confuse workspace name with collection/item IDs** in adaptor code — credential lookups need the actual *workspace* name, not a STAC collection ID. Passing the wrong one produces a misleading "no credentials found for workspace X" error where X is actually a collection ID.

8. **Test end-to-end** — see [Test commercial data adaptor changes](test-commercial-data-adaptor.md). Confirm both that the workflow succeeds *and* that the purchase itself completes (a successful workflow run doesn't guarantee a successful purchase), and confirm `order:status` progresses past `pending` with assets landing under the `commercial-data/` prefix.
