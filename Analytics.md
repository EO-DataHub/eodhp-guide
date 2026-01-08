# EODH Analytics Guide

## Google Analytics Integration
The application utilizes Google Analytics 4 (GA4) to track user interactions and page navigation. The implementation is handled via the `react-ga4` library, which provides a clean abstraction over the native GA4 API and ensures proper integration with the React lifecycle.

## Configuration requirements
To enable analytics, the measurement ID must be provided via environment variables. If this variable is missing, the initialization logic will abort, allowing the application to run in environments without tracking enabled.

- `VITE_GA_MEASUREMENT_ID`: The unique measurement ID provided by the GA4 dashboard (format: `G-XXXXXXXXXX`).

## Implementation Strategy
Tracking is centralized in the `useGoogleAnalytics` hook. This implementation uses the **Observer Pattern** by subscribing directly to the Router instance. This approach decouples tracking from the component render tree and ensures accurate page view capture on every route change or main interaction.



## Inactive Users
Configuration guide for the "Inactive Users" audience in GA4. This metric identifies users who have visited the platform historically but have recorded 0 sessions in the last 30 days.

### Audience Configuration
Navigate to **Admin** -> **Data display** -> **Audiences** and select **Create a custom audience**.

- **Name**: `Inactive Users (30 Days)`
- **Description**: `Users who have previously visited the platform but have not returned (0 sessions) in the last 30 days.`

### Logic Definition
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

### Membership Settings
- Set **Membership duration** to "Set to maximum limit" (ensures users remain in the inactive list indefinitely until they return).

> **Note:** GA4 provides client-side estimates. For precise inactivity data, we need to evaluate the use of Keycloak's `last_login_time` value.



## Create Custom Dimensions in GA4
The events sent by EODH include several custom parameters (e.g. `collection_id`, `providers`, `data_type`) that are not available in GA4 by default.  
To use them in **Explorations, filters, and breakdowns**, they must be registered as **Custom Dimensions** in Google Analytics.

### Steps
1. Go to **GA4 → Admin → Property settings → Data Display → Custom Definitions**
2. Click **Create custom dimension**
3. Configure the dimension:
   - **Dimension name:** (e.g. *Collection ID*)
   - **Scope:** `Event`
   - **Event parameter:** Must exactly match the parameter name sent from EODH useGoogleAnalyticsActions.ts
4. Save the dimension
5. Repeat for each parameter you want to analyze

> **Note:** After creation, dimensions may take a few minutes to appear in **Explore**, and up to 24 hours to be available in standard GA4 reports.

### Recommended Custom Dimensions
The following table lists the recommended event parameters to register as custom dimensions based on the analytics events implemented in the application:

| Dimension Name        | Event Parameter     | Scope | Description |
|----------------------|---------------------|-------|-------------|
| Collection ID        | `collection_id`     | Event | Unique identifier of the collection |
| Collection Title     | `collection_title`  | Event | Human-readable collection name |
| Asset Key            | `asset_key`         | Event | Asset identifier within the item or collection |
| Asset Roles          | `roles`             | Event | Roles associated with the asset (e.g. thumbnail, data) |
| Data Type            | `data_type`         | Event | Indicates whether the data is `open` or `commercial` |
| Providers            | `providers`         | Event | Data provider(s) associated with the collection |
| Is Preview           | `is_preview`        | Event | Indicates if the downloaded asset is a preview |
| Auth Status          | `auth_status`       | Event | User authentication status (`logged_in` or `guest`) |

> It is recommended to start with the most relevant dimensions (e.g. *Collection ID*, *Data Type*, *Providers*) and add others incrementally as reporting needs evolve.



## Handling `(not set)` Values in GA4 Explorations
When building dashboards in GA4 Explorations, you may notice a line or category labeled **`(not set)`**.  
This value appears when an event does not include the parameter associated with the selected dimension (e.g. `collection_title`), or when events occurred before the custom dimension was created.

This is expected behavior in GA4 and does not indicate an error in the application.

To keep dashboards clean and focused on meaningful data, `(not set)` values can be excluded using filters.

### Option 1: Excluding `(not set)` Explicitly
1. Go to **Explore → Open your Exploration**
2. In the middle column, scroll down to **Settings → Filters**
3. Click **“Drop or select dimension or metric”**
4. Select the relevant dimension (e.g. **Collection Title**)
   - If the dimension is not available, add it first from **Variables → Dimensions → +**
5. Configure the filter:
   - **Exclude**
   - **Dimension:** Collection Title
   - **Condition:** exactly matches
   - **Value:** `(not set)`
6. Click **Apply**

This removes `(not set)` entries from the chart and legend.


### Option 2: Filtering by Event Name (Recommended)
A common cause of `(not set)` values is mixing multiple event types (e.g. `page_view`, `session_start`) with custom events in the same exploration.

To avoid this, filter the exploration to include only the event you want to analyze.
1. Go to **Explore → Open your Exploration**
2. In **Settings → Filters**, click **“Drop or select dimension or metric”**
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


## Exploration: Users Who Downloaded Open Imagery
This exploration shows the **number of unique users** who downloaded imagery from the Resource Catalog, considering **both item-level and collection-level downloads**, and filtered to **open data only**.


### Objective
Measure:
- **Unique users** (not total downloads)
- Who triggered any imagery download event
- From **open data collections only**

This exploration aggregates:
- `download_item_imagery`
- `download_collection_imagery`


### Dimensions
Add the following dimensions:
- **Event name**
- **Downloaded Imagery Data Type** (`data_type`)

### Metrics
Add the following metric:
- **Total users**

> This metric counts unique users who triggered at least one matching event during the selected time range.

### Filters Configuration
To ensure correct results, configure **two separate filters**, each using a different dimension.

#### Filter 1: Download Events Only
This filter ensures only imagery download events are included.

- **Include**
- **Dimension:** Event name
- **Condition:** matches regex
- **Value:** download_(item|collection)_imagery

> This aggregates both item-level and collection-level download events.

#### Filter 2: Open Data Only
This filter restricts results to open data downloads.

- **Include**
- **Dimension:** Downloaded Imagery Data Type
- **Condition:** exactly matches
- **Value:** open

> Value is based on useGoogleAnalyticsActions.ts data_type options.


## Exploration: Number of Open Images Downloaded by Collection
This exploration shows the **total number of images downloaded** from the Resource Catalog, filtered to **open data only**, and **disaggregated by collection**.  
It includes **both item-level and collection-level downloads**.


### Objective
Measure:
- **Total number of images downloaded** (not unique users)
- From **open data only**
- Broken down **by collection**

This exploration aggregates:
- `download_item_imagery`
- `download_collection_imagery`


### Dimensions
Add the following dimensions:
- **Collection Title**
- **Event name**
- **Downloaded Imagery Data Type** (`data_type`)


### Metrics
Add the following metric:
- **Event count**

> This metric represents the total number of download actions (each download counts as one image).


### Filters Configuration
To ensure correct results, configure **two separate filters**, each using a different dimension.

#### Filter 1: Download Events Only
This filter ensures only imagery download events are included.

- **Include**
- **Dimension:** Event name
- **Condition:** matches regex
- **Value:** download_(item|collection)_imagery

> This aggregates both item-level and collection-level download events.

#### Filter 2: Open Data Only
This filter restricts results to open data downloads.

- **Include**
- **Dimension:** Downloaded Imagery Data Type
- **Condition:** exactly matches
- **Value:** open

> Value is derived from the `data_type` parameter sent by `useAnalyticsActions.ts`.


## Exploration: Purchased Imagery (Commercial Data)
This exploration shows the **number of purchased images** from the Resource Catalog, based on **commercial imagery purchases**, and allows inspection of key purchase attributes per event.

### Objective
Measure:
- **Imagery purchase activity** from the RC
- Based exclusively on the `purchase_imagery` event
- With full visibility of **item, collection, provider, bundle, and workspace**

This exploration is **event-based**, not page-based.

### Dimensions
Add the following dimensions:
- **Event name**
- **Item ID**
- **Collection ID**
- **Provider**
- **Product Bundle**
- **Workspace**

> All dimensions are **event-scoped custom dimensions** mapped directly to event parameters sent by the application.

### Metrics
Add one of the following metrics depending on the analysis goal:
- **Event count** → total number of purchases  
- **Total users** → number of unique users who made at least one purchase  

> Use **Event count** for volume analysis and **Total users** for user-level analysis.


### Filters Configuration: Purchase Events Only
This filter ensures that only purchase-related events are included.
- **Include**
- **Dimension:** Event name
- **Condition:** exactly matches
- **Value:** `purchase_imagery`

> With this filter applied, the exploration reflects **only commercial imagery purchases**.


## Exploration: Page Engagement Time by Session
This exploration shows **how much time users spend on each page**, measured as **average engagement time per session**, helping identify the most and least engaging pages in the application.

### Objective
Measure:
- **Average engagement time**
- Per **page**
- Aggregated **by session**

This exploration helps answer:
- Which pages keep users engaged longer
- Which pages have low engagement and may require UX or content improvements

### Dimensions
Add the following dimension:
- **Page path + query string**

> Use **Page path** instead if you want to group URLs without query parameters and reduce noise.

### Metrics
Add the following metric:
- **Average engagement time per session**

> This is the official GA4-supported metric for time-based engagement analysis.

### Visualization
- **Table**

### Notes & Limitations
- GA4 Explorations **do not support min/max calculations** for engagement time.
- “Engagement time” is **not available** as a standalone metric in Explore.
- For min/max or raw time analysis, **BigQuery export is required**.


## Exploration: User Journey Through RC
This exploration visualizes the **user navigation path through the Resource Catalog**, showing the **next action or page** users take after each interaction.

It helps answer:
- What users do after landing on RC pages
- How users move between collections, items, and actions
- Where users drop off or continue deeper into the catalog

This exploration is built using **GA4 Path Exploration** create a new exploration of type **Path exploration**.

### Objective
Measure and understand:
- The **sequence of pages or events** users follow
- The **next click / next step** in a user session
- Common navigation paths within the Resource Catalog

### Starting Point Configuration
Choose **one** of the following starting point options depending on the analysis goal.

#### Option A: Start from Page View (General Navigation)
Use this option to analyze the overall navigation flow across the site.

- **Starting point**
- **Dimension:** Event name
- **Value:** `page_view`

This answers:
> “After landing on any page, what do users do next?”

#### Option B: Start from Resource Catalog Entry
Use this option to focus specifically on the **Resource Catalog user journey**.

- **Starting point**
- **Dimension:** Page path + query string
- **Select values such as:**
  - `/`
  - `/finder`
  - `/finder/*`

This limits the journey to interactions happening **inside the Resource Catalog**.


## Exploration: Average Engagement Time per Session (Global)
This exploration shows the **average time users spend per session across all pages** of the application, providing a high-level view of overall user engagement.


### Objective
Measure:
- **Average engagement time per session**
- Across **all pages**
- Aggregated at the **session level**

This helps answer:
- How long users typically stay active during a session
- Whether overall engagement is improving or declining
- The impact of UX or feature changes on session depth

### Dimensions
No dimensions are required for a global view.

Optional (for segmentation):
- **Date**
- **Auth status** (a custom dimension need to be created)

> Adding dimensions will split the metric; leave empty for a single global value.

### Notes and Limitations
- This metric represents an **average across all sessions**
- It does **not reflect individual user behavior**
- GA4 does not support min/max calculations for engagement time in Explorations
