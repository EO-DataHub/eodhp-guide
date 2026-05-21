---
title: Google Analytics (GA4) reference
doc_status: ok
last_reviewed: 2026-05-20
reviewed_by: geodowd
review_notes: "Split from legacy docs/Analytics.md"
tags:
  - analytics
---

# Google Analytics (GA4) reference

Lookup facts for frontend GA4 instrumentation and reporting configuration. Procedures live under [Analytics how-to guides](../how-to/analytics/index.md); integration rationale is in [Google Analytics integration](../explanation/analytics-google-analytics-integration.md).

## Configuration

To enable analytics, the measurement ID must be provided via environment variables. If this variable is missing, the initialization logic will abort, allowing the application to run in environments without tracking enabled.

| Variable | Description |
|---------|-------------|
| `VITE_GA_MEASUREMENT_ID` | The unique measurement ID from the GA4 dashboard (format: `G-XXXXXXXXXX`). |

## Recommended custom dimensions

The events sent by EODH include custom parameters that are not available as dimensions in GA4 until registered. Parameter names **must exactly match** those sent from the application (see `useGoogleAnalyticsActions.ts`).

| Dimension Name | Event Parameter | Scope | Description |
|----------------|-----------------|-------|-------------|
| Collection ID | `collection_id` | Event | Unique identifier of the collection |
| Collection Title | `collection_title` | Event | Human-readable collection name |
| Asset Key | `asset_key` | Event | Asset identifier within the item or collection |
| Asset Roles | `roles` | Event | Roles associated with the asset (e.g. thumbnail, data) |
| Data Type | `data_type` | Event | Indicates whether the data is `open` or `commercial` |
| Providers | `providers` | Event | Data provider(s) associated with the collection |
| Is Preview | `is_preview` | Event | Indicates if the downloaded asset is a preview |
| Auth Status | `auth_status` | Event | User authentication status (`logged_in` or `guest`) |

Starting with dimensions such as *Collection ID*, *Data Type*, and *Providers* is recommended; add others as reporting needs evolve.

## Related reporting events (examples)

Procedures use these identifiers in explorations:

| Events / notes |
|----------------|
| `download_item_imagery`, `download_collection_imagery` |
| `purchase_imagery` |
| `view_collection`, `page_view`, `session_start` |
