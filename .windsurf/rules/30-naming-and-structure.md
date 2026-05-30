---
trigger: always_on
description: Naming conventions and repository structure
---

# Naming & Structure

Consistency is a feature. Follow these conventions exactly.

## Repository layout

```
/                       repo root
├── .windsurf/          agent rules and workflows (this directory)
├── models/             transformation models (dbt or equivalent)
│   ├── staging/        1:1 with sources, light cleaning, renamed
│   ├── intermediate/   reusable business logic, not exposed
│   └── marts/          consumer-facing, contracted
├── pipelines/          orchestration (Airflow DAGs, Dagster assets, etc.)
├── sources/            source definitions and ingestion configs
├── tests/              custom data tests not covered by the framework
├── macros/             reusable transformation snippets
├── docs/               ADRs, runbooks, onboarding
├── infra/              IaC (Terraform, Pulumi, etc.)
└── seeds/              static reference data
```

## Table naming

| Layer        | Pattern                        | Example                       |
|--------------|--------------------------------|-------------------------------|
| Source       | `src_<system>__<entity>`       | `src_salesforce__account`     |
| Staging      | `stg_<system>__<entity>`       | `stg_salesforce__account`     |
| Intermediate | `int_<domain>__<purpose>`      | `int_orders__joined_lines`    |
| Fact         | `fct_<grain>`                  | `fct_order_lines`             |
| Dimension    | `dim_<entity>`                 | `dim_customer`                |
| Aggregate    | `agg_<grain>_<metric>`         | `agg_daily_revenue`           |
| Snapshot     | `snap_<entity>`                | `snap_customer_status`        |

- All lowercase. Words separated by underscores.
- Double underscore (`__`) separates a *system* from an *entity*, never use it
  inside a single name.
- No abbreviations unless they are universal in the business (e.g. `sku`,
  `usd`). When in doubt, spell it out.

## Column naming

- Surrogate keys: `<entity>_sk` (e.g. `customer_sk`).
- Natural/business keys: `<entity>_id` (e.g. `customer_id`).
- Booleans: prefixed `is_` or `has_` (e.g. `is_active`, `has_subscription`).
- Timestamps: suffixed `_at` (e.g. `created_at`, `updated_at`).
- Dates: suffixed `_date` (e.g. `order_date`).
- Counts: suffixed `_count`.
- Amounts: suffixed with currency or unit (`_usd`, `_kg`).
- Never use SQL reserved words as column names.

## Pipeline / job naming

- DAG / asset names: `<domain>__<purpose>` (e.g. `finance__daily_revenue`).
- Tasks within a DAG: verb-led (`extract_orders`, `validate_invoices`).

## Branches and PRs

- Feature branches: `feat/<ticket>-<short-slug>`.
- Schema changes: `schema/<ticket>-<short-slug>`.
- PRs must include: change summary, affected models, migration plan (if any),
  rollback plan (if any), and links to lineage diffs.
