---
title: GA4 exploration — page engagement time by session
doc_status: ok
last_reviewed: 2026-05-20
reviewed_by: geodowd
review_notes: "Split from legacy docs/Analytics.md"
tags:
  - analytics
---

# GA4 exploration — page engagement time by session

This exploration shows **how much time users spend on each page**, measured as **average engagement time per session**, helping identify the most and least engaging pages in the application.

## Objective

Measure:

- **Average engagement time**
- Per **page**
- Aggregated **by session**

This exploration helps answer:

- Which pages keep users engaged longer
- Which pages have low engagement and may require UX or content improvements

## Dimensions

Add the following dimension:

- **Page path + query string**

> Use **Page path** instead if you want to group URLs without query parameters and reduce noise.

## Metrics

Add the following metric:

- **Average engagement time per session**

> This is the official GA4-supported metric for time-based engagement analysis.

## Visualization

- **Table**

## Notes and limitations

- GA4 Explorations **do not support min/max calculations** for engagement time.
- “Engagement time” is **not available** as a standalone metric in Explore.
- For min/max or raw time analysis, **BigQuery export is required**.
