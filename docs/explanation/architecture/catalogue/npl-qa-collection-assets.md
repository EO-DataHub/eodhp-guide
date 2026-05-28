---
title: NPL QA Assessments as STAC Collection Assets
doc_status: ok
tags:
  - stac
  - catalogue
  - qa
  - npl
  - data-quality
last_reviewed:
reviewed_by:
review_notes: Drafted from issue #144. Issue #164 (OC data QA) is a placeholder pending NPL check-in.
---

# NPL QA Assessments as STAC Collection Assets

Explains how NPL quality assessments are structured, how to deposit them so they are picked up automatically, and the current state of implementation.

---

## What the QA assessments are

NPL produces two types of quality assessment for data collections in the EODH catalogue:

| Type | STAC asset key | Description | Frequency |
|------|---------------|-------------|-----------|
| Quality Processes Review (QPR) | `qa_documentation` | Assesses methodology, documentation, and processes used to produce the data | One per collection |
| Radiometric uncertainty | `qa_radiometric` | Measures sensor calibration and radiometric accuracy over time | One consolidated file per collection (time-series data merged into a single JSON) |

Both are JSON files produced by NPL workflows running in the NPL workspace.

## Collections covered

QPR and radiometric uncertainty files currently exist for:

| Collection | EODH catalogue name | QPR | Radiometric uncertainty |
|------------|-------------------|-----|------------------------|
| Sentinel-2 L1C | `sentinel2_l1c` | yes | yes |
| Airbus Pleiades HR | `airbus_phr_data` | yes | yes |
| Airbus Pleiades Neo | `airbus_pneo_data` | yes | no |
| Airbus SPOT | `airbus_spot_data` | yes | no |
| PlanetScope Scenes | `PSScene` | yes | yes |
| SkySat Collect | `SkySatCollect` | yes | no |

**Note on Sentinel-2:** The NPL QA records assess Sentinel-2 L1C data, but the EODH catalogue currently contains Sentinel-2 ARD rather than L1C.

---

## How QA assets are attached to STAC collections

The `harvest-transformer` service handles attaching QA assets to STAC collection records. When a collection is processed, the transformer:

1. Takes the collection's STAC `id` field.
2. Looks up the `id` in the **collection map** (see below) to find the corresponding QA key. If no entry exists, the collection `id` is used as the QA key directly.
3. Constructs expected URLs for both QA file types using the QA key and checks whether each file exists (HTTP HEAD request).
4. Adds a STAC asset entry (`qa_documentation` and/or `qa_radiometric`) for each file that exists.

Assets are only ever added, never overwritten — if a collection already has a `qa_documentation` or `qa_radiometric` asset, it is left unchanged.

---

## Depositing QA files: what NPL needs to do

### S3 bucket structure

QA files must be deposited in the `collection-qa` S3 bucket at:

```
s3://collection-qa.s3.eu-west-2.amazonaws.com/
  qa_documentation/
    {qa_key}_qa_check_quality_processes_review.json
  qa_radiometric/
    {qa_key}_qa_check_radiometric_unc_all_dates.json
```

Where `{qa_key}` is the key assigned to the collection in the collection map (see below). If no map entry exists, `{qa_key}` is the collection's STAC `id`.

### File naming

| Asset type | File name pattern |
|------------|------------------|
| QPR | `{qa_key}_qa_check_quality_processes_review.json` |
| Radiometric uncertainty | `{qa_key}_qa_check_radiometric_unc_all_dates.json` |

Examples for a collection with qa_key `airbus_phr_data`:

- `qa_documentation/airbus_phr_data_qa_check_quality_processes_review.json`
- `qa_radiometric/airbus_phr_data_qa_check_radiometric_unc_all_dates.json`

Earlier iterations produced per-year radiometric files (e.g. `..._2022-12-01_2022-12-31.json`). These have been consolidated into single per-collection files. **The transformer expects the consolidated single-file format only.**

### File content

The transformer does not validate or parse the contents of the QA JSON files — it only checks that they exist. The files may contain any valid JSON structure. No specific schema is enforced at ingestion time.

### The collection map

The collection map is a JSON file at:

```
s3://collection-qa.s3.eu-west-2.amazonaws.com/qa-collection-map.json
```

It maps STAC collection IDs to QA keys:

```json
{
  "sentinel2_ard": "sentinel-2_l1c"
}
```

This mapping is needed when the QA file prefix differs from the STAC collection `id` — for example, when NPL's files use `sentinel-2_l1c` as a prefix but the EODH catalogue collection is called `sentinel2_ard`.

**To add a new collection:** add an entry to `qa-collection-map.json` and deposit the corresponding QA files in the correct prefixes. No front-end changes are needed.

### When will the asset appear in the catalogue?

QA assets are attached to a collection the next time that collection is harvested. Depositing a file in S3 does not immediately update the catalogue — the change takes effect on the next scheduled harvest run for that collection.

CEDA and Planet collections are harvested daily; Airbus collections are harvested monthly. For Airbus collections, a file deposited shortly after a harvest run could wait up to a month before appearing.

CEDA catalogue data is ingested via the configuration harvester — a CronJob that scans the `stac-harvester-configurations` GitHub repository for changes daily at 06:00 UTC and triggers the STAC ingestion pipeline. Planet uses a standard Kubernetes CronJob. Airbus jobs are triggered via Argo Events (calendar EventSource → Sensor → Kubernetes Job).

| Harvester | Collections | Frequency | Prod (UTC) | Staging (UTC) | Test (UTC) | Trigger |
|-----------|-------------|-----------|------------|---------------|------------|---------|
| CEDA (config harvester) | CEDA catalogue | Daily | 06:00 | 06:00 | 06:00 | Kubernetes CronJob |
| Planet | `PSScene`, `SkySatCollect` | Daily | 07:30 | 07:30 | 07:30 | Kubernetes CronJob |
| Airbus SAR | `airbus_sar_data` | Monthly | 20th 02:30 | 16th 13:20 | 13th 15:00 | Argo Events |
| Airbus SPOT | `airbus_spot_data` | Monthly | 20th 03:00 | 16th 13:20 | 13th 15:00 | Argo Events |
| Airbus PNEO | `airbus_pneo_data` | Monthly | 20th 03:30 | 16th 13:20 | 13th 15:00 | Argo Events |
| Airbus PHR | `airbus_phr_data` | Monthly | 20th 04:00 | 16th 13:20 | 13th 15:00 | Argo Events |

---

## Resulting STAC asset structure

Once the transformer runs, the collection record gains assets such as:

```json
"assets": {
  "qa_documentation": {
    "href": "https://collection-qa.s3.eu-west-2.amazonaws.com/qa_documentation/airbus_phr_data_qa_check_quality_processes_review.json",
    "type": "application/json",
    "title": "Quality Processes Review",
    "roles": ["metadata", "quality"]
  },
  "qa_radiometric": {
    "href": "https://collection-qa.s3.eu-west-2.amazonaws.com/qa_radiometric/airbus_phr_data_qa_check_radiometric_unc_all_dates.json",
    "type": "application/json",
    "title": "Radiometric Uncertainty Assessment",
    "roles": ["metadata", "quality"]
  }
}
```

---

## How the front end uses QA assets

The RC UI reads `qa_documentation` and `qa_radiometric` from the STAC collection's `assets` field and renders them in the Collection Info side panel, under two tabs:

- **Document QA** (`qa_documentation`) — fetches the JSON and renders a colour-coded table of quality assessment results across categories (product information, metrology, product generation), with ratings such as Not Assessed, Basic, Good, Excellent, and Ideal. Links referenced in the results are extracted and rendered below the table.
- **Product Validation** (`qa_radiometric`) — fetches the JSON and renders per-band radiometric uncertainty with pass/fail/partial-pass status. Users can select different time periods from the consolidated file.

If either asset href is missing from the collection record, or the fetch fails after one retry, the tab shows a "not available for this collection" message. The UI does not fall back to the old NPL workspace path.

---

### Manual handover process

NPL is currently depositing QA JSON files by attaching them to GitHub issues rather than writing them directly to the S3 bucket. The preferred end-state is for NPL to deposit files directly to `s3://collection-qa` from their Hub workspace. This is planned for a future phase.
