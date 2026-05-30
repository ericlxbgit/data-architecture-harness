---
trigger: model_decision
description: Common anti-patterns in data architecture. Pull in when reviewing existing code, refactoring, evaluating proposed designs, or when a request smells like one of these patterns.
---

# Anti-Patterns

When you see any of these, name it and propose the alternative. Do not
silently replicate the pattern even if "the rest of the codebase does it."

## Modeling

**One Big Table for everything.** A 200-column denormalized table that every
team reads from. Looks fast, becomes unmaintainable. *Fix*: split into marts
by domain; share dimensions; let BI denormalize on read.

**Implicit grain.** A table where no one can answer "what does one row mean?"
*Fix*: declare grain in the description, enforce with a `unique` test.

**Reusing column names with different semantics.** `revenue` means net in one
table and gross in another. *Fix*: name the semantics (`revenue_net_usd`,
`revenue_gross_usd`).

**Modeling at consumption time.** Building the same metric ten different ways
in ten different dashboards. *Fix*: define metrics in the semantic layer or
in agreed-on marts.

**Premature denormalization.** Flattening dimensions into facts because "it's
faster." *Fix*: keep the star; let materialization choices handle performance.

## Pipelines

**The cron + insert anti-pattern.** Scheduled blind inserts with no
idempotency. First retry duplicates rows. *Fix*: partition-replace or merge.

**Manual interventions in the runbook.** Steps that read "if X fails, log in
and rerun task Y." *Fix*: encode that recovery in the orchestrator.

**Time-based downstream dependencies.** Mart job runs at 03:00 "because ingest
usually finishes by 02:30." *Fix*: data-aware scheduling.

**One giant DAG.** Every pipeline in one DAG so dependencies "just work."
*Fix*: split by domain, link via assets/sensors.

**Logic in the scheduler.** Business rules inside the orchestrator config.
*Fix*: business logic lives in transformations or services, not in the DAG.

## Schema

**Rename in place.** Renaming a column and breaking every downstream consumer.
*Fix*: add new, deprecate old, drop later.

**Type widening to dodge errors.** Casting everything to `STRING` because the
loader keeps failing. *Fix*: fix the loader; types are a feature.

**Nullable everything.** Making every column nullable to avoid ingest errors.
*Fix*: enforce nullability where the business invariant requires it.

## Security

**PII spread by convenience.** Copying a PII column into a sandbox "just for
this analysis." *Fix*: synthetic data or masked extracts.

**Role-by-individual.** Grants assigned to people instead of groups. *Fix*:
groups + IaC.

**Logged everything for debugging.** PII or secrets in logs because debugging
was hard. *Fix*: structured logs with explicit redaction; debug with sampled
masked data.

## Process

**Tests disabled to ship.** Disabling a failing test because the deadline
matters more than correctness. *Fix*: fix the data or the test; never both.

**The hero rebuild.** One person rebuilds the warehouse over a weekend with
no review. *Fix*: incremental change, reviewed.

**Documentation in a separate wiki.** Model docs live anywhere but next to
the code. They are stale within a month. *Fix*: docs in `schema.yml`.

**Lineage-by-comment.** "This feeds into the finance dashboard" written as a
SQL comment. *Fix*: real lineage via the transformation tool.
