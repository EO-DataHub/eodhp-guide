---
title: Sample Data Ingestion Process
doc_status: ok
tags:
  - stac
  - data-catalogues
last_reviewed:
reviewed_by:
review_notes: Migrated from docs/operations/sample-data-ingest/ingestion.md; fixtures in sample-data-ingest/samples/.
---
# Sample Data Ingestion Process

This guide describes the process for ingesting sample STAC catalog data into the EODHP system. The workflow involves uploading STAC metadata files to S3, triggering a harvest process via Pulsar messages, and copying the actual data files.

## Prerequisites

- Access to the Kubernetes cluster
- AWS CLI configured with appropriate credentials
- Pulsar client tools installed locally
- `kubectl` configured to access the cluster
- `jq` installed for JSON processing

## Overview

The ingestion process consists of four main steps:

1. **Upload STAC metadata files** to the catalogue S3 bucket
2. **Send a Pulsar message** to trigger the harvest/ingest process
3. **Copy data files** to the workspace S3 bucket
4. **Create `.s3keep` files** to ensure directories are visible in Jupyter notebooks

---

## Step 1: Upload STAC Metadata Files

Example STAC payloads for this workflow live next to this page under **`sample-data-ingest/samples/`**:

- **Sub-catalog:** [samples/airbus.json](sample-data-ingest/samples/airbus.json)
- **Collection:** [samples/airbus_phr.json](sample-data-ingest/samples/airbus_phr.json)
- **Item:** [samples/item1.json](sample-data-ingest/samples/item1.json)

There can be multiple files of each type. Upload them (or your own equivalents) to the `catalogue-population-eodhp` S3 bucket in the folder structure described below.

### S3 Path Structure

For a workspace named `exampleworkspace` in the production environment, the S3 paths should follow this pattern:

**Sub-catalog:**
```
s3://catalogue-population-eodhp/file-harvester/exampleworkspace-eodhp-config/catalogs/user/catalogs/exampleworkspace/catalogs/commercial-data/catalogs/airbus.json
```

**Collection:**
```
s3://catalogue-population-eodhp/file-harvester/exampleworkspace-eodhp-config/catalogs/user/catalogs/exampleworkspace/catalogs/commercial-data/catalogs/airbus/collections/airbus_phr.json
```

**Item:**
```
s3://catalogue-population-eodhp/file-harvester/exampleworkspace-eodhp-config/catalogs/user/catalogs/exampleworkspace/catalogs/commercial-data/catalogs/airbus/collections/airbus_phr/items/item1.json
```

**Note:** Replace `exampleworkspace` with your actual workspace name in all paths.

### STAC record content and transformer behaviour

The harvest-transformer normalises STAC before ingestion. When preparing your STAC files, the following applies.

**Where records end up**
- **Placement is determined by the S3 path (folder structure), not by the `collection` field in the JSON.** Use the folder structure you want in the catalogue (e.g. `…/collections/<collection_id>/items/<item_id>.json`). The `collection` property in the item should match that collection id for correctness but does not control where the file is written.

**Links**
- You can send **empty or missing `links`**: the transformer adds/rewrites `root` and `self` with the correct EODH catalogue URLs.
- It **rewrites** these if present: `child`, `collection`, `item`, `items`, `parent`, `root`, `self`.
- It **does not add** `parent` or `collection` when missing—it only rewrites them if present. For STAC best practice, include placeholders and they will be overwritten, e.g.:
  ```json
  "links": [
    {"rel": "self", "href": "https://example.com/collections/my_coll/items/my_item"},
    {"rel": "root", "href": "https://example.com/"},
    {"rel": "parent", "href": "https://example.com/collections/my_coll"},
    {"rel": "collection", "href": "https://example.com/collections/my_coll"}
  ]
  ```

**What to include**

| Aspect | Include? | Notes |
|--------|----------|-------|
| Path / folder structure | Yes | Dictates where the record ends up in the catalogue. |
| `collection` (property) | Yes | Should match the collection id in the path. |
| Links (self, root, parent, collection) | Optional | Can be blank; catalogue-structure links are added or rewritten. |
| Item content | Yes | `id`, `type`, `stac_version`, `geometry`, `bbox`, `properties`, `assets`, etc. |
| License | As needed | Valid SPDX id triggers license links; otherwise provide as appropriate. |

---

## Step 2: Send Pulsar Message to Trigger Harvest

### 2.1 Set up Port Forwarding

First, forward the Pulsar service ports from the cluster to your local machine:

```bash
kubectl port-forward service/pulsar-proxy -n pulsar 6650:6650 8080:8080
```

Keep this terminal session running while you send messages.

### 2.2 Create the Harvest Message

Create a JSON file (e.g., `message.json`) with the following structure:

```json
{
    "id": "harvester/workspace_file_harvester/exampleworkspace",
    "workspace": "exampleworkspace",
    "repository": "",
    "branch": "",
    "bucket_name": "catalogue-population-eodhp",
    "source": "exampleworkspace-eodhp-config/",
    "target": "",
    "added_keys": [
        "file-harvester/exampleworkspace-eodhp-config/catalogs/user/catalogs/exampleworkspace/catalogs/commercial-data/catalogs/airbus.json",
        "file-harvester/exampleworkspace-eodhp-config/catalogs/user/catalogs/exampleworkspace/catalogs/commercial-data/catalogs/airbus/collections/airbus_phr.json",
        "file-harvester/exampleworkspace-eodhp-config/catalogs/user/catalogs/exampleworkspace/catalogs/commercial-data/catalogs/airbus/collections/airbus_phr/items/item1.json"
    ],
    "updated_keys": [],
    "deleted_keys": []
}
```

**Important:** Update the following fields to match your workspace:
- `id`: Replace `exampleworkspace` with your workspace name
- `workspace`: Replace `exampleworkspace` with your workspace name
- `source`: Replace `exampleworkspace-eodhp-config/` with your workspace prefix
- `added_keys`: List all S3 paths to the STAC files you uploaded in Step 1 *but remove the leading `s3://catalogue-population-eodhp/` from each key*.

### 2.3 Send the Message

The Pulsar client requires the message to be on a single line. Use these commands to format and send the message:

```bash
# Convert JSON to single-line format
jq -c . message.json > /tmp/one-line.json

# Send the message to Pulsar
bin/pulsar-client produce persistent://public/default/harvested \
  -f /tmp/one-line.json
```

**Note:** Ensure the `pulsar-client` binary is in your `PATH`, or invoke it with its full install path.

---

## Step 3: Copy Data Files

After the metadata has been harvested, copy the actual data files from the sample data bucket to the workspace bucket:

```bash
aws s3 cp --recursive \
  s3://sample-data-bucket/airbus/airbus_phr_data/ \
  s3://workspaces-eodhp/exampleworkspace/commercial-data/airbus/airbus_phr_data/
```

**Note:** Adjust the source and destination paths for your specific data layout and workspace name.

---

## Step 4: Create `.s3keep` Files for Jupyter Visibility

When copying files using `aws s3 cp`, empty directories are not created in S3, which means they will not appear in Jupyter notebooks unless you place an object under them. Create `.s3keep` sentinel objects in directories you need visible.

Either:

1. **Manually create `.s3keep` objects** for each directory in the S3 console, or  
2. **Use a script** to create `.s3keep` in every directory touched by uploads.

Example:

```bash
# List all directories and create .s3keep files
aws s3 ls s3://workspaces-eodhp/exampleworkspace/commercial-data/airbus/airbus_phr_data/ --recursive | \
  awk '{print $4}' | \
  xargs -I {} dirname {} | \
  sort -u | \
  xargs -I {} aws s3 cp /dev/null s3://workspaces-eodhp/exampleworkspace/commercial-data/airbus/airbus_phr_data/{}/.s3keep
```

---

## Troubleshooting

- **Port forwarding issues:** Confirm cluster access and that the `pulsar-proxy` Service is reachable.
- **Pulsar client not found:** Confirm the Apache Pulsar client is installed on your workstation and callable from the terminal.
- **S3 upload failures:** Confirm AWS credentials and write access on the catalogue and workspace buckets.
- **Harvest not triggering:** Confirm the compact JSON message landed on topic `persistent://public/default/harvested` with the paths you uploaded.

For Pulsar dashboards and backlog checks in Grafana, see [Monitor resources with Grafana](../observability-logging/monitor-resources.md).
