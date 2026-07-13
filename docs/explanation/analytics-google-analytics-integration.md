---
title: Google Analytics integration (GA4)
doc_status: ok
tags:
  - analytics
last_reviewed: 2026-05-20
reviewed_by: geodowd
review_notes: "Split from legacy docs/Analytics.md"
---

# Google Analytics integration (GA4)

## Google Analytics Integration

The application utilizes Google Analytics 4 (GA4) to track user interactions and page navigation. The implementation is handled via the `react-ga4` library, which provides a clean abstraction over the native GA4 API and ensures proper integration with the React lifecycle.

For enabling measurement in deployments, configuration is documented in [**Google Analytics reference**](../reference/analytics-google-analytics.md).

## Implementation Strategy

Tracking is centralized in the `useGoogleAnalytics` hook. This implementation uses the **Observer Pattern** by subscribing directly to the Router instance. This approach decouples tracking from the component render tree and ensures accurate page view capture on every route change or main interaction.

For GA audience setup, dimensions, explorations, and reporting recipes see [Analytics how-to guides](../how-to/analytics/index.md).
