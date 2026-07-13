---
title: GA4 exploration — unique users who downloaded open imagery
doc_status: ok
last_reviewed: 2026-05-20
reviewed_by: geodowd
review_notes: "Split from legacy docs/Analytics.md"
tags:
  - analytics
---

# GA4 exploration — unique users who downloaded open imagery

This exploration shows the **number of unique users** who downloaded imagery from the Resource Catalog, considering **both item-level and collection-level downloads**, and filtered to **open data only**.

**Reference:** [Google Analytics (GA4) reference](../../reference/analytics-google-analytics.md)

## Objective

Measure:

- **Unique users** (not total downloads)
- Who triggered any imagery download event
- From **open data collections only**

This exploration aggregates:

- `download_item_imagery`
- `download_collection_imagery`

## Dimensions

Add the following dimensions:

- **Event name**
- **Downloaded Imagery Data Type** (`data_type`)

## Metrics

Add the following metric:

- **Total users**

> This metric counts unique users who triggered at least one matching event during the selected time range.

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

> Value is based on `useGoogleAnalyticsActions.ts` `data_type` options.
