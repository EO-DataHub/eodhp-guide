---
title: Exclude `(not set)` in GA4 Explorations
doc_status: ok
last_reviewed: 2026-05-20
reviewed_by: geodowd
review_notes: "Split from legacy docs/Analytics.md"
---

# Exclude `(not set)` in GA4 Explorations

When building dashboards in GA4 Explorations, you may notice a line or category labeled **`(not set)`**.

This value appears when an event does not include the parameter associated with the selected dimension (e.g. `collection_title`), or when events occurred before the custom dimension was created.

This is expected behavior in GA4 and does not indicate an error in the application.

To keep dashboards clean and focused on meaningful data, `(not set)` values can be excluded using filters.

## Option 1: Excluding `(not set)` explicitly

1. Go to **Explore → Open your Exploration**
2. In the middle column, scroll down to **Settings → Filters**
3. Click **"Drop or select dimension or metric"**
4. Select the relevant dimension (e.g. **Collection Title**)
   - If the dimension is not available, add it first from **Variables → Dimensions → +**
5. Configure the filter:
   - **Exclude**
   - **Dimension:** Collection Title
   - **Condition:** exactly matches
   - **Value:** `(not set)`
6. Click **Apply**

This removes `(not set)` entries from the chart and legend.

## Option 2: Filtering by event name (recommended)

A common cause of `(not set)` values is mixing multiple event types (e.g. `page_view`, `session_start`) with custom events in the same exploration.

To avoid this, filter the exploration to include only the event you want to analyze.

1. Go to **Explore → Open your Exploration**
2. In **Settings → Filters**, click **"Drop or select dimension or metric"**
3. Select **Event name**
4. Configure the filter:
   - **Include**
   - **Dimension:** Event name
   - **Condition:** exactly matches
   - **Value:** `view_collection`
5. Click **Apply**

This ensures the exploration only includes the selected event and prevents `(not set)` values from appearing due to unrelated events.

Once applied, `(not set)` entries will be removed from the chart and legend, resulting in a clearer and more accurate visualization.

> **Note:** GA4 does not retroactively populate missing parameters for past events.  
> Even if all new events include the required parameters, historical data may still contain `(not set)` values and should be filtered out as described above.
