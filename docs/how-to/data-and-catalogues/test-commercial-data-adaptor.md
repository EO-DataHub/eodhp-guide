---
title: Test commercial data adaptor changes
doc_status: unreviewed
last_reviewed:
reviewed_by:
review_notes:
tags:
  - commercial-data
  - workflows
---
# Test commercial data adaptor changes

How to safely test changes to commercial data adaptor code without affecting live orders.

**See also:** [Commercial Data Purchasing Pipeline](../../explanation/architecture/commercial-data-purchasing.md) for an overview of how the full purchase flow works.

## Steps

1. **Create a new workflow in the data provider workspace** (e.g. `airbus`)
   - Copy the original adaptor's CWL file.
   - Change the workflow ID to avoid overriding the existing workflow used in production purchasing.
   - This allows you to test without affecting live orders.

2. **Test the workflow**
   - For the `airbus-optical-adaptor` workflow, use [this example input](test-commercial-data-adaptor/example_input.json).
   - The workflow must be executed in the `airbus` workspace using an `airbus` API key.
   - The workspace specified in the JSON input must have an API key configured to access the Airbus purchasing API.

3. **Verify the purchase succeeded**
   - **Important:** The workflow can succeed even if the purchase failed.
   - Check the STAC item to confirm the purchase status (`order:status` field).
   - A successful workflow execution does not guarantee a successful purchase.

4. **Make incremental changes**
   - Workflow error messages often don't reveal the root cause.
   - Make small, incremental changes to make debugging easier.
   - Test after each change to isolate issues.
