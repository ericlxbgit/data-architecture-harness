---
description: Scaffold a new data pipeline end-to-end (source → staging → marts → tests → DAG)
---

# Workflow: New Pipeline

Use this when adding a new data source or building a new end-to-end pipeline.

## Inputs to gather first

Before scaffolding anything, ask the user (in one message, not one at a time):

1. **Source system** — what's the upstream system, and how do we read it?
   (Fivetran, API, file drop, CDC stream?)
2. **Business owner** — who owns the data and signs off on the contract?
3. **Grain** — what does one row in the eventual mart represent?
4. **Freshness SLA** — how stale can the data be before someone complains?
5. **PII expectations** — does this source carry Tier 3+ data?
6. **Volume expectations** — rough rows/day or GB/day, for materialization
   and partitioning choices.

If any of these are unclear, stop and ask. Do not guess.

## Steps

1. **Source declaration** in `sources/`:
   - Add the source with `loaded_at_field` and a `freshness` block.
   - Document the source system and its owner.

2. **Staging model** in `models/staging/<system>/`:
   - One file per source table, `stg_<system>__<entity>.sql`.
   - Explicit column selection, casts, renames. No joins.
   - Adjacent `schema.yml` with descriptions and basic tests.

3. **Intermediate models** in `models/intermediate/<domain>/`:
   - Only if the logic is reused by more than one mart.
   - `ephemeral` or `view` unless performance forces otherwise.

4. **Mart model** in `models/marts/<domain>/`:
   - Declare grain in the description.
   - PK enforced by `unique` + `not_null`.
   - FKs tested with `relationships`.
   - Tag PII columns per `70-security-pii.md`.
   - Enforce contract: `config: { contract: { enforced: true } }`.

5. **Tests**:
   - All mandatory tests from `80-testing-quality.md`.
   - Custom singular tests for any business invariant the user described.

6. **Orchestration** in `pipelines/`:
   - DAG / asset named `<domain>__<purpose>`.
   - Data-aware dependencies, not time-based.
   - Retry policy + timeout per `60-pipelines.md`.

7. **Runbook** in `docs/runbooks/`:
   - How to backfill.
   - Common failure modes and their fixes.
   - On-call owner.

8. **PR checklist** in the description:
   - Grain declared.
   - PII classified.
   - Contract enforced.
   - Lineage diff attached.
   - Backfill plan if applicable.

## When done

Summarize what was created, what tests will run on the first PR build, and
which downstream consumers (if any) need to be notified before merge.
