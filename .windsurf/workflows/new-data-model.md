---
description: Add a single new model (fact, dimension, or intermediate) to an existing pipeline
---

# Workflow: New Data Model

Use this for adding a single model — not a whole new pipeline.

## Inputs to confirm before writing SQL

1. **Layer** — staging, intermediate, fact, dimension, aggregate?
2. **Grain** — exact, in plain English. "One row per X per Y."
3. **Sources** — which upstream models? Confirm they exist.
4. **Consumer** — who will read this and for what?
5. **Materialization** — view, table, incremental? Default per
   `50-data-modeling.md`, deviate only with reason.
6. **PII** — any classified columns flowing in?

## Steps

1. **File placement** — correct folder per `30-naming-and-structure.md`.

2. **Naming** — correct prefix (`stg_`, `int_`, `fct_`, `dim_`, `agg_`)
   per the layer.

3. **SQL structure**:
   - CTEs in dependency order, last one is named `final`.
   - `select * from final` at the bottom.
   - Explicit columns at every layer. No `SELECT *` except the last
     `select * from final`.
   - Comments only where the *why* is non-obvious. The SQL should read.

4. **Schema entry** in the adjacent `schema.yml`:
   - Description starts with the grain on line 1.
   - Owner in `meta`.
   - PII classification per `70-security-pii.md` for each sensitive column.
   - Freshness SLA in `meta`.
   - Contract enforced for marts.

5. **Tests** per `80-testing-quality.md`:
   - Grain test (unique + not_null on PK).
   - Referential tests on FKs.
   - Range / accepted-values where applicable.
   - Custom test for any business invariant.

6. **Materialization choice**:
   - Volume small, freshness loose → `view`.
   - Volume medium, reads frequent → `table`.
   - Volume large, append-only or partition-replaceable → `incremental`.
   - For incremental: declare `unique_key` and `partition_by`.

7. **Lineage check** — run the transformation tool's lineage command
   locally to confirm the new model has the expected upstream/downstream
   edges before committing.

## What to push back on

- "Just denormalize it into the fact" — keep the star unless there's a
  measured performance reason. Cite `99-antipatterns.md`.
- "Skip the test for now" — refuse. Cite `80-testing-quality.md`.
- "Put it in marts directly without staging" — refuse. Source columns
  must pass through staging for type / rename / cleaning.
