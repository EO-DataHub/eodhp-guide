---
title: GA4 exploration — average engagement time per session (global)
doc_status: ok
last_reviewed: 2026-05-20
reviewed_by: geodowd
review_notes: "Split from legacy docs/Analytics.md"
tags:
  - analytics
---

# GA4 exploration — average engagement time per session (global)

This exploration shows the **average time users spend per session across all pages** of the application, providing a high-level view of overall user engagement.

**Reference:** [Google Analytics (GA4) reference](../../reference/analytics-google-analytics.md) (for registering **Auth status** if used as a custom dimension).

## Objective

Measure:

- **Average engagement time per session**
- Across **all pages**
- Aggregated at the **session level**

This helps answer:

- How long users typically stay active during a session
- Whether overall engagement is improving or declining
- The impact of UX or feature changes on session depth

## Dimensions

No dimensions are required for a global view.

Optional (for segmentation):

- **Date**
- **Auth status** (a custom dimension needs to be created)

> Adding dimensions will split the metric; leave empty for a single global value.

## Metrics

Add:

- **Average engagement time per session**

## Notes and limitations

- This metric represents an **average across all sessions**
- It does **not reflect individual user behavior**
- GA4 does not support min/max calculations for engagement time in Explorations
