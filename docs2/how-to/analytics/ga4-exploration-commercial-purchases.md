---
title: GA4 exploration — purchased imagery (commercial data)
doc_status: ok
last_reviewed: 2026-05-20
reviewed_by: geodowd
review_notes: "Split from legacy docs/Analytics.md"
---

# GA4 exploration — purchased imagery (commercial data)

This exploration shows the **number of purchased images** from the Resource Catalog, based on **commercial imagery purchases**, and allows inspection of key purchase attributes per event.

**Reference:** [Google Analytics (GA4) reference](../../reference/analytics-google-analytics.md)

## Objective

Measure:

- **Imagery purchase activity** from the RC
- Based exclusively on the `purchase_imagery` event
- With full visibility of **item, collection, provider, bundle, and workspace**

This exploration is **event-based**, not page-based.

## Dimensions

Add the following dimensions:

- **Event name**
- **Item ID**
- **Collection ID**
- **Provider**
- **Product Bundle**
- **Workspace**

> All dimensions are **event-scoped custom dimensions** mapped directly to event parameters sent by the application.

## Metrics

Add one of the following metrics depending on the analysis goal:

- **Event count** → total number of purchases  
- **Total users** → number of unique users who made at least one purchase  

> Use **Event count** for volume analysis and **Total users** for user-level analysis.

## Filters configuration: purchase events only

This filter ensures that only purchase-related events are included.

- **Include**
- **Dimension:** Event name
- **Condition:** exactly matches
- **Value:** `purchase_imagery`

> With this filter applied, the exploration reflects **only commercial imagery purchases**.
