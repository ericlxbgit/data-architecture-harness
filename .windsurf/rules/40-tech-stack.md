---
trigger: always_on
description: The actual tech stack for this project — FILL IN before use
---

# Tech Stack

> **FILL THIS IN** for your project. The agent will assume this context for
> every decision. Be specific — versions matter for SQL dialects, operators,
> and library APIs.

## Warehouse / lakehouse

- **Engine**: <!-- e.g. BigQuery / Snowflake / Databricks / Redshift / DuckDB -->
- **Dialect quirks the agent should know**:
  - <!-- e.g. "BigQuery: prefer `SAFE_CAST`, use `_TABLE_SUFFIX` for sharded tables" -->
  - <!-- e.g. "Snowflake: prefer `QUALIFY`, use `COPY GRANTS` on view replaces" -->

## Transformation layer

- **Tool**: <!-- e.g. dbt Core 1.8 / dbt Cloud / SQLMesh / Dataform -->
- **Materialization defaults**:
  - Staging: `view`
  - Intermediate: `ephemeral` or `view`
  - Marts: `table` or `incremental`
- **Test framework**: <!-- e.g. dbt tests + dbt_expectations -->

## Orchestration

- **Tool**: <!-- e.g. Airflow 2.9 / Dagster / Prefect / Argo / Cron -->
- **Scheduling conventions**:
  - <!-- e.g. "All daily jobs run at 02:00 UTC after raw ingest completes" -->
- **Sensors / triggers**:
  - <!-- e.g. "Use data-aware scheduling; do not use time-based for downstream marts" -->

## Ingestion

- **Tools**: <!-- e.g. Fivetran, Airbyte, custom Python, Debezium for CDC -->
- **CDC strategy**: <!-- e.g. "Append-only with `_loaded_at`; we resolve current state in staging" -->

## Storage

- **Lake / object store**: <!-- e.g. GCS, S3, ADLS -->
- **File formats**: <!-- e.g. Parquet for landing, Iceberg for curated -->

## BI / consumption

- **Tools**: <!-- e.g. Looker, Mode, Hex, Metabase, custom -->
- **Semantic layer**: <!-- e.g. dbt Semantic Layer, Cube, LookML -->

## Catalog / governance

- **Catalog**: <!-- e.g. DataHub, OpenMetadata, Atlan, Unity Catalog -->
- **PII tagging**: <!-- e.g. "Tagged at column level via dbt meta + propagated to catalog" -->

## Infrastructure

- **IaC**: <!-- e.g. Terraform 1.7, modules in `infra/modules/` -->
- **CI/CD**: <!-- e.g. GitHub Actions, runs `dbt build --select state:modified+` on PRs -->

## Environments

| Env     | Purpose                          | Naming                           |
|---------|----------------------------------|----------------------------------|
| dev     | Per-developer sandbox            | `<warehouse>__dev_<username>`    |
| staging | Pre-prod integration             | `<warehouse>__staging`           |
| prod    | Production                       | `<warehouse>__prod`              |

## Constraints the agent must respect

- <!-- e.g. "Region pinned to EU; never spin up resources in US." -->
- <!-- e.g. "Daily warehouse spend cap is $X; flag any query expected to exceed Y TB scanned." -->
- <!-- e.g. "All PII columns must be hashed before leaving the curated layer." -->
