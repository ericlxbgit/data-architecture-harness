---
trigger: always_on
description: Non-negotiable data architecture principles
---

# Architectural Principles

These are the principles you apply by default. Deviations require an explicit
justification in the PR description.

## 1. Single source of truth (SSoT)

Every business entity has exactly one authoritative table. Downstream models
read from it; no parallel "shadow" versions. If two tables disagree, that is a
bug, not a feature.

## 2. Grain is declared, not inferred

Every model has a documented grain ("one row per customer per day", "one row
per order line"). The primary key enforces it. No model is merged without a
declared grain.

## 3. Idempotency by default

Running the same pipeline twice with the same inputs must produce the same
outputs. No accidental duplicates, no time-dependent side effects. Use
`MERGE`/upserts or full-refresh + partition-replace, never blind `INSERT`.

## 4. Immutable raw, mutable models

The raw/landing layer is append-only and untouched. All cleaning, conforming,
and business logic happens in transformations that can be rebuilt from raw at
any time.

## 5. Schema-on-write for curated layers

The raw layer may be schema-on-read. The curated (silver/gold, marts) layers
are strictly typed and schema-validated. Untyped JSON does not leak into marts.

## 6. Data contracts at boundaries

Any table consumed by another team, dashboard, or service has an explicit
contract: column names, types, nullability, semantics, freshness SLA, and
owner. Breaking changes require versioning, not in-place mutation.

## 7. Lineage is a first-class artifact

Every transformation declares its sources. Lineage must be machine-readable
(via the transformation tool, not buried in comments). A consumer must be able
to trace any column in a mart back to its source within minutes.

## 8. Fail loud, fail early

Pipelines fail closed, not open. A broken upstream blocks the downstream
mart rather than serving stale-but-silent data. Tests run *before* publish,
not after.

## 9. Reversibility

Every change is reversible: migrations are forward + backward, deploys are
rollback-safe, deletes are soft where business rules allow. Irreversible
operations require a written approval trail.

## 10. Cost is a design constraint, not an afterthought

Partition keys, clustering, materialization choice (view vs table vs
incremental), and refresh cadence are design decisions made at modeling time,
not retrofitted when the bill arrives.

## 11. Separation of compute, storage, and orchestration

Don't couple them. The orchestrator does not know SQL dialects; the warehouse
does not schedule jobs; storage is independent of who reads it.

## 12. Documentation lives with the code

Model descriptions, column comments, and ownership tags live in the
transformation files (e.g. `dbt` `schema.yml`), not in a separate wiki that
will go stale.
