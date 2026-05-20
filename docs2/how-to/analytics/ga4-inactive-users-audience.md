---
title: Create an inactive users audience in GA4
doc_status: ok
last_reviewed: 2026-05-20
reviewed_by: geodowd
review_notes: "Split from legacy docs/Analytics.md"
---

# Create an inactive users audience in GA4

Configuration guide for the "Inactive Users" audience in GA4. This metric identifies users who have visited the platform historically but have recorded 0 sessions in the last 30 days.

**Reference:** [Google Analytics (GA4) reference](../../reference/analytics-google-analytics.md)

## Audience configuration

Navigate to **Admin** -> **Data display** -> **Audiences** and select **Create a custom audience**.

- **Name**: `Inactive Users (30 Days)`
- **Description**: `Users who have previously visited the platform but have not returned (0 sessions) in the last 30 days.`

## Logic definition

The audience requires two specific condition groups:

**1. Include Condition (The Base)**  
Defines users who have visited at least once in history.

- Section: "Include users when"
- Event: `session_start`
- Parameter: `Event count` > `0`
- Time period: "At any point in time"

**2. Exclude Condition (The Inactivity)**  
Removes users who have visited recently.

- Section: "Add condition group to exclude" -> "Exclude users temporarily"
- Event: `session_start`
- Parameter: `Event count` > `0`
- Time period settings:
    - Toggle Time Period: **ON**
    - Duration: `30 Days`
    - Logic: "True in the most recent period"

## Membership settings

- Set **Membership duration** to "Set to maximum limit" (ensures users remain in the inactive list indefinitely until they return).

> **Note:** GA4 provides client-side estimates. For precise inactivity data, we need to evaluate the use of Keycloak's `last_login_time` value.
