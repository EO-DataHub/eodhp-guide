---
title: Catalogue Browser Search Form Pipeline
doc_status: unreviewed
tags:
  - rc-ui
  - stac
  - cql2
  - json-schema
  - search
last_reviewed:
reviewed_by:
review_notes:
---

# Catalogue Browser Search Form Pipeline

Explains the `json-schema-forms` module in the RC UI — how it selects a collection-specific schema, renders a filter form, and translates user input into a CQL2 payload for the STAC search API.

The pipeline has three distinct stages: schema selection → form rendering → query building.

---

## Stage 1 — Schema selection (`rjsfCollections.ts`)

`rjsfCollections` is a static registry that maps collection ID strings to their JSON Schema definitions. All collection schemas are bundled at build time as JSON module imports.

```kroki-plantuml
@startuml
skinparam defaultTextAlignment center
skinparam roundcorner 8

(selectedCollection.id) -right-> (rjsfCollections[id]) : lookup
(rjsfCollections[id]) -right-> (JSON Schema object) : resolve
note bottom of (rjsfCollections[id])
  falls back to
  rjsfCollections.default
end note
@enduml
```

`FilterPanel.tsx` passes the resolved schema to `QueryForm`. The registry is keyed by collection ID (e.g., `"sentinel1"`, `"PSScene"`, `"cmip6"`) and is the coupling point between STAC catalogue metadata and the form system.

!!! note
    `rjsfCollections.ts` contains a comment acknowledging this should eventually be server-driven rather than hard-coded. The `$id` URIs on schemas indicate the intended direction is fetching schemas from a server, allowing new collections to be added without a frontend release.

---

## Stage 2 — Form rendering (`QueryForm.tsx` + JSON Schemas)

### Schema dereferencing

Collection schemas use **JSON Schema `$ref` pointers** to compose reusable field definitions from versioned base schemas. For example:

```json
{
  "eo:cloud_cover": {
    "$ref": "schemas/eo/v1.1.0/form_schema.json#/properties/eo:cloud_cover"
  }
}
```

`QueryForm` resolves these at runtime using `@apidevtools/json-schema-ref-parser` with an in-memory resolver supplying local schema file contents. The resolved (dereferenced) schema is what RJSF uses to render the form.

### Base schema library (`assets/schemas/`)

Versioned base schemas define reusable field primitives, grouped by STAC Extension namespace:

| Schema | Namespace | Fields |
|---|---|---|
| `eodh/v1.0.0` | _(core)_ | `start_datetime`, `end_datetime` |
| `eo/v1.1.0`, `eo/v2.0.0` | `eo:` | `cloud_cover`, `snow_cover` |
| `sar/v1.0.0` | `sar:` | `polarizations`, `instrument_mode`, `product_type`, `observation_direction` |
| `sat/v1.0.0` | `sat:` | `orbit_state`, `platform_international_designator` |
| `view/v1.0.0` | `view:` | `incidence_angle`, `sun_azimuth`, `sun_elevation`, `azimuth` |
| `cmip6/v1.0.0` | `cmip6:` | `variable_id`, `frequency`, `source_id`, `experiment_id`, etc. |

Schemas carry a non-standard `operator` field (e.g., `"operator": "<="`) alongside standard JSON Schema keywords. This field is passed through RJSF transparently and consumed at query-build time to determine the CQL2 comparison operator.

### UI schema and custom widgets

`form-ui.json` is the RJSF `uiSchema` configuration. It maps field names to custom widget components:

| Widget name | Component | Used for |
|---|---|---|
| `RangeWidget` | `CustomRangeWidget` (Radix UI `Slider`) | Numeric threshold fields — cloud cover, incidence angle |
| `SelectWidget` | `CustomSelectWidget` (Radix UI `Select`) | `anyOf` enum fields |
| `DateWidget` | `CustomDateWidget` (MUI DatePicker) | Date fields |
| `TextWidget` | `CustomTextWidget` | Read-only text display |

`RangeWidget` reads `schema.minimum`, `schema.maximum`, and `schema.operator` to determine slider bounds and whether the handle renders as a minimum threshold (inverted) or maximum threshold.

---

## Stage 3 — Query building (`queries/cql2.ts` + namespace dispatch handlers)

On form submit, RJSF emits `formData` along with the live `schema`. `QueryForm` calls:

```ts
buildQuery(d.schema, d.formData)
```

### Form flattening

`flattenForm` recursively walks the nested `formData` object and emits flat key-value pairs, joining parent and child keys with `:`:

```
{ eo: { "cloud_cover": 30 } }  →  { "eo:cloud_cover": 30 }
```

This converts RJSF's nested structure into the flat key space the dispatch table expects, and discards `null`/`undefined` entries (fields the user left blank).

### Namespace dispatch

`cql2.ts` defines a `dispatchTable` keyed by namespace prefix:

```ts
const dispatchTable = {
  '': custom,    // date ranges, incidence_angle, clear_percent
  eo: eo,
  sar: sar,
  sat: sat,
  view: view,
  cmip6: cmip6,
  ukcp: ukcp18,
  cordex: cordex,
  sentinel1: sentinel1,
  planet: planet,
}
```

`buildFilter` iterates every namespace, calling `buildQueryParams` for each. Each namespace module exports a `QueryDispatch` object: a map of field-key → `(schema, obj) => QueryCondition`. The `schema` argument gives handlers access to the `operator` field to determine the CQL2 comparison operator at runtime.

### Special cases

**SAR polarizations** (`sarQuery.ts`): `sar:polarizations` is an array, but CQL2 only supports `in` for array membership. `buildPolarizationsFilter` constructs an explicit AND of `in` conditions for selected polarizations combined with a NOT of excluded ones — enforcing an exact match rather than a subset match.

**Planet datetimes** (`planetQuery.ts`): Planet STAC items store `datetime` at the top level (not under `properties`), and the time component must be appended explicitly (`T00:00:00.000Z` / `T23:59:59.999Z`).

**Climate datasets** (CMIP6, UKCP18, CORDEX): These use `start_datetime_range` / `end_datetime_range` instead of `start_datetime` / `end_datetime`. The `customQuery.ts` handlers map these to CQL2 filters against `properties.end_datetime` and `properties.start_datetime` — an interval overlap query rather than a point-in-time query.

**CORDEX/UKCP18 → STAC property mapping**: These handlers map namespaced form keys (e.g., `cordex:frequency`) to the actual STAC property names in the index (e.g., `properties.time_frequency`), which differ from the form field names.

### Output structure

```json
{
  "fields": { "include": ["properties"] },
  "filter": {
    "op": "and",
    "args": [ "...one QueryCondition per populated field..." ]
  },
  "sortby": [
    { "field": "properties.end_datetime", "direction": "desc" }
  ]
}
```

Planet schemas sort by `"acquired"` at the top level rather than `"properties.end_datetime"`, detected by checking `schema.$id` against known Planet schema IDs.

---

## Data flow

```kroki-plantuml
@startuml
skinparam backgroundColor white
skinparam roundcorner 8

start

:FilterPanel passes **selectedCollection.id**;

:Lookup schema in **rjsfCollections**\n(falls back to //default// if not found);

:Pass JSON Schema (containing **$ref**s)\nto **QueryForm**;

:Resolve **$ref**s at runtime\n//json-schema-ref-parser// + in-memory resolver\nMerge **form-ui.json** uiSchema;

:Render **RJSF <Form>**\nwith custom widgets\n(RangeWidget · SelectWidget · DateWidget · TextWidget);

:User fills and submits form\n→ //{ schema, formData }//;

partition "**buildQuery**(schema, formData)" {
  :""flattenForm(formData)""\n→ flat colon-namespaced key-value pairs;
  split
    :""buildFilter(schema, flat)""\nDispatch each key to namespace handler\n→ [ QueryCondition, … ];
  split again
    :""buildSort(isPlanet)""\n→ sortby clause;
  split end
  :Assemble CQL2 payload\n""{ fields, filter, sortby }"";
}

:Returned to FilterPanel\n→ POST to STAC search API;

stop
@enduml
```

---

## Key design decisions

**Schema as the source of truth**: The `operator` field embedded in JSON Schemas drives comparison logic at query-build time, keeping field semantics co-located with their definitions rather than scattered across query handlers.

**Namespace-prefixed form keys as routing**: The colon-delimited key convention (e.g., `eo:cloud_cover`) mirrors the STAC Extension namespacing convention and doubles as the dispatch table key, making the mapping between form fields and query handlers self-documenting.

**`$ref` composition**: Collection schemas are thin overlays that override titles, defaults, and `anyOf` enumerations on top of shared base schemas. This avoids duplicating field type/range definitions while still allowing per-collection customisation.

**Static bundle, aspirationally dynamic**: Schemas are compiled into the bundle at build time. The `$id` URIs on schemas indicate the intended direction is fetching schemas from a server, which would allow new collections to be added without a frontend release.
