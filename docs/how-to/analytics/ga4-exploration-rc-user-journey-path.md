---
title: GA4 path exploration — user journey through the Resource Catalog
doc_status: ok
last_reviewed: 2026-05-20
reviewed_by: geodowd
review_notes: "Split from legacy docs/Analytics.md"
tags:
  - analytics
---

# GA4 path exploration — user journey through the Resource Catalog

This exploration visualizes the **user navigation path through the Resource Catalog**, showing the **next action or page** users take after each interaction.

It helps answer:

- What users do after landing on RC pages
- How users move between collections, items, and actions
- Where users drop off or continue deeper into the catalog

This exploration is built using **GA4 Path Exploration**: create a new exploration of type **Path exploration**.

## Objective

Measure and understand:

- The **sequence of pages or events** users follow
- The **next click / next step** in a user session
- Common navigation paths within the Resource Catalog

## Starting point configuration

Choose **one** of the following starting point options depending on the analysis goal.

### Option A: Start from page view (general navigation)

Use this option to analyze the overall navigation flow across the site.

- **Starting point**
- **Dimension:** Event name
- **Value:** `page_view`

This answers:

> “After landing on any page, what do users do next?”

### Option B: Start from Resource Catalog entry

Use this option to focus specifically on the **Resource Catalog user journey**.

- **Starting point**
- **Dimension:** Page path + query string
- **Select values such as:**
  - `/`
  - `/finder`
  - `/finder/*`

This limits the journey to interactions happening **inside the Resource Catalog**.
