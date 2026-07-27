---
title: Collection Categorisation
doc_status: unreviewed
tags:
  - rc-ui
  - stac
last_reviewed:
reviewed_by:
review_notes: Derived from eodhp-rc-ui source (src/constants/collectionCategorizations.ts).
---

# Collection Categorisation

Explains how the RC UI classifies STAC collections into facets (data category, resolution, provider, location, time period, licence, QA availability) for the quick-filter and group-by controls on the collection gallery page.

**Reference:** [Resource Catalogue UI](../../reference/services/resource-catalogue-ui.md)

---

## How it works

Categorisation is metadata *about* a collection that isn't necessarily present in the STAC collection record itself. It is hand-maintained per collection ID in:

```
eodhp-rc-ui/src/constants/collectionCategorizations.ts
```

The file defines a `collectionCategorization` object keyed by collection ID. Each entry assigns the collection to zero or more values across seven facets:

| Facet | Example values |
|-------|-----------------|
| `dataCategory` | Optical & Multispectral, Radar, Land Cover, Climate Data, Climate Projections, Hyperspectral |
| `resolution` | Less than 1m, 1m–<10m, 10m–<100m, ... Greater than 100km |
| `provider` | Airbus, ESA, CEDA, UK Met Office, Planet, Open Cosmos, ... |
| `location` | Global, Continental, United Kingdom |
| `timePeriod` | Historic, Current, Forecast |
| `license` | Commercial Data, Open Data, My Data |
| `qa` | Not part of the static table — computed at runtime (see [QA availability](#qa-availability-is-computed-not-static)) |

Each facet (except `qa`) can hold **multiple values** — e.g. `sentinel1` lists both `Provider.ESA` and `Provider.CEDA` — or an **empty array**, which means the collection is treated as "Uncategorised" for that facet.

A collection with no entry in `collectionCategorization` at all (checked via `isValidCollectionId`) is uncategorised on every facet, except that it can still match the `lic:my-data` filter if it's a user collection.

---

## QA availability is computed, not static

Unlike the other six facets, `qa` isn't set per collection in the table — it's computed at request time by `collectionHasQAIndicators` (`src/library/collectionQuality.ts`), which checks whether the collection actually has quality-assessment data available. `qa:available` / `qa:unavailable` filter and group-by values are derived from that check rather than looked up from `collectionCategorization`.

---

## Where categorisation is used

- **`QuickFilterMenu`** (`src/components/QuickFilterMenu/QuickFilterMenu.tsx`) — renders one `Select` per facet, populated from the `dataCategories`, `resolutions`, `providers`, `locations`, `timePeriods`, `licenses`, `qualityAssessments` option lists exported alongside the categorisation table.
- **`useCollections`** (`src/hooks/useCollections.ts`) — applies the active `QuickFilter` to the collection list (`filteredCollections`), and, when a `groupBy` facet is selected, buckets collections into `groupedByCollections` using `filterMapping[groupBy]` as the ordered list of group headings.
- **`FilterTags`** and **`CollectionCard`** — display the active filters and per-collection facet values on the gallery page.
- **`ItemCard`** — uses a separate `itemCardFieldsToDisplay` map, keyed by `DataCategory`, to decide which extra STAC item properties to surface for a given category (e.g. climate projection items show `variable_long_name`, `frequency`, `scenario`, `start_datetime`, `end_datetime`).

---

## Adding or updating a collection's categorisation

1. Add or edit the collection's entry in `collectionCategorization` in `src/constants/collectionCategorizations.ts`, using the existing `DataCategory`/`Resolution`/`Provider`/`Location`/`TimePeriod`/`License` enums (add a new enum member first if none fits).
2. Leave a facet as `[]` if the collection genuinely doesn't fit any existing value — it will show up under "Uncategorised" for that facet rather than being silently dropped from the gallery.
3. `qa` does not need to be set — it's derived automatically from the item metadata.
4. Rebuild/redeploy the RC UI; there is no server-side or STAC catalogue change required, since this is frontend-only configuration.
