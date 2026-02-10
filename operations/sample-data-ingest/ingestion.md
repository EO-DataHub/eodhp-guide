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

In the samples data folder, you'll find examples of:
- **STAC sub-catalog** files (e.g., `airbus.json`)
- **Collection** files (e.g., `airbus_phr.json`)
- **Item** files (e.g., `item1.json`)

There can be multiple files of each type. These need to be uploaded to the `catalogue-population-eodhp` S3 bucket in the correct folder structure.

### S3 Path Structure

For a workspace named `exampleworkspace` is the production environment, the S3 paths should follow this pattern:

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
- `added_keys`: List all S3 paths to the STAC files you uploaded in Step 1 *but you must remove the leading `s3://catalogue-population-eodhp/` from the URL*.

### 2.3 Send the Message

The Pulsar client requires the message to be on a single line. Use these commands to format and send the message:

```bash
# Convert JSON to single-line format
jq -c . message.json > /tmp/one-line.json

# Send the message to Pulsar
bin/pulsar-client produce persistent://public/default/harvested \
  -f /tmp/one-line.json
```

**Note:** Ensure the `pulsar-client` binary is in your PATH or provide the full path to it.

---

## Step 3: Copy Data Files

After the metadata has been harvested, copy the actual data files from the sample data bucket to the workspace bucket:

```bash
aws s3 cp --recursive \
  s3://sample-data-bucket/airbus/airbus_phr_data/ \
  s3://workspaces-eodhp/exampleworkspace/commercial-data/airbus/airbus_phr_data/
```

**Note:** Adjust the source and destination paths to match your specific data location and workspace name.

---

## Step 4: Create `.s3keep` Files for Jupyter Visibility

When copying files using `aws s3 cp`, empty directories are not created in S3, which means they won't be visible in Jupyter notebooks. To ensure directories are visible, you need to create `.s3keep` files in each directory.

You can do this by either:

1. **Manually creating `.s3keep` files** in the S3 console for each directory
2. **Using a script** to automatically create `.s3keep` files in all directories

Example script to create `.s3keep` files:

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

- **Port forwarding issues**: Ensure you have the correct cluster access and the Pulsar service is running
- **Pulsar client not found**: Check that the Pulsar client tools are installed and in your PATH
- **S3 upload failures**: Verify your AWS credentials have write access to the target buckets
- **Harvest not triggering**: Check that the Pulsar message was sent successfully and verify the topic name
