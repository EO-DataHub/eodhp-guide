---
title: GA4 exploration — open images downloaded by collection
doc_status: ok
last_reviewed: 2026-05-20
reviewed_by: geodowd
review_notes: "Split from legacy docs/Analytics.md"
---

# GA4 exploration — open images downloaded by collection

This exploration shows the **total number of images downloaded** from the Resource Catalog, filtered to **open data only**, and **disaggregated by collection**.

It includes **both item-level and collection-level downloads**.

**Reference:** [Google Analytics (GA4) reference](../../reference/analytics-google-analytics.md)

## Objective

Measure:

- **Total number of images downloaded** (not unique users)
- From **open data only**
- Broken down **by collection**

This exploration aggregates:

- `download_item_imagery`
- `download_collection_imagery`

## Dimensions

Add the following dimensions:

- **Collection Title**
- **Event name**
- **Downloaded Imagery Data Type** (`data_type`)

## Metrics

Add the following metric:

- **Event count**

> This metric represents the total number of download actions (each download counts as one image).

## Filters configuration

Configure **two separate filters**, each using a different dimension.

### Filter 1: Download events only

This filter ensures only imagery download events are included.

- **Include**
- **Dimension:** Event name
- **Condition:** matches regex
- **Value:** download_(item|collection)_imagery

> This aggregates both item-level and collection-level download events.

### Filter 2: Open data only

This filter restricts results to open data downloads.

- **Include**
- **Dimension:** Downloaded Imagery Data Type
- **Condition:** exactly matches
- **Value:** open

> Value is derived from the `data_type` parameter sent by `useGoogleAnalyticsActions.ts`.
