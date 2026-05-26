---
title: Register custom dimensions in GA4
doc_status: ok
last_reviewed: 2026-05-20
reviewed_by: geodowd
review_notes: "Split from legacy docs/Analytics.md"
tags:
  - analytics
---

# Register custom dimensions in GA4

The events sent by EODH include several custom parameters (e.g. `collection_id`, `providers`, `data_type`) that are not available in GA4 by default.

To use them in **Explorations, filters, and breakdowns**, they must be registered as **Custom Dimensions** in Google Analytics.

**Reference:** [Recommended custom dimensions table](../../reference/analytics-google-analytics.md#recommended-custom-dimensions)

## Steps

1. Go to **GA4 → Admin → Property settings → Data Display → Custom Definitions**
2. Click **Create custom dimension**
3. Configure the dimension:
   - **Dimension name:** (e.g. *Collection ID*)
   - **Scope:** `Event`
   - **Event parameter:** Must exactly match the parameter name sent from EODH `useGoogleAnalyticsActions.ts`
4. Save the dimension
5. Repeat for each parameter you want to analyze

> **Note:** After creation, dimensions may take a few minutes to appear in **Explore**, and up to 24 hours to be available in standard GA4 reports.
