---
trigger: glob
globs: ["models/**", "**/*.sql", "**/schema.yml", "**/schema.yaml"]
description: Data modeling conventions (dimensional / vault / wide)
---

# Data Modeling

## Modeling approach

This project uses **Kimball dimensional modeling** in the marts layer.
> Change this line if you use Data Vault, Activity Schema, One Big Table, or
> a hybrid — and remove the rules below that don't apply.

## Staging layer rules

- One staging model per source table. No joins in staging.
- Cast every column to its final type.
- Rename columns to the project's snake_case convention.
- Apply only "true" cleaning: trimming, casing, null-replacement of sentinels.
  No business logic.
- Always select columns explicitly. Never `SELECT *` from a source.

## Intermediate layer rules

- Used to factor out logic that appears in more than one mart.
- Never exposed to consumers. Never documented as a contract.
- Materialized as `ephemeral` or `view` unless performance forces otherwise.

## Fact tables

- Declare grain in the model description, in plain English, on line 1.
- Primary key enforces the grain. Add a `unique` test on it.
- Foreign keys to dimensions use surrogate keys (`<dim>_sk`), not natural keys.
- Additive measures preferred. Semi-additive and non-additive measures must be
  flagged in the model description.
- Always include the source system grain (e.g. `order_id`) as a degenerate
  dimension for traceability, even if not used in BI.

## Dimension tables

- One row per entity. `unique` and `not_null` tests on the surrogate key.
- Surrogate key is generated, not natural — use a stable hash of the natural
  key + source system, not a row number.
- Type 2 SCDs use `valid_from`, `valid_to`, `is_current` (boolean).
  `valid_to` is `NULL` (preferred) or a far-future sentinel — pick one and
  stay consistent across all SCD2 dims.
- Include an "unknown" / "-1" row in every dimension to support left-side
  joins from facts without losing rows.

## Schema evolution

- Adding a nullable column: safe, ship it.
- Adding a non-nullable column: requires a default or backfill in the same PR.
- Removing a column: add a `deprecated` tag, wait one release, then drop.
- Changing semantics of an existing column without changing the name: **never**.
  Always introduce a new column with a new name.

## Tests every model must have

- `unique` and `not_null` on the primary key (or grain columns).
- `relationships` on every foreign key.
- Freshness check on source tables.
- Volume anomaly test on critical facts (row count within expected band).

## Documentation every model must have

In `schema.yml`:

```yaml
- name: fct_order_lines
  description: |
    One row per order line. Grain: order_id + line_number.
    Source of truth for line-level revenue. Replaces legacy
    `revenue_detail` (deprecated 2025-Q4).
  config:
    contract: { enforced: true }
    meta:
      owner: data-platform@company.com
      pii: false
      sla_freshness_hours: 6
  columns:
    - name: order_line_sk
      description: Surrogate key. Hash of order_id + line_number.
      tests: [unique, not_null]
```
