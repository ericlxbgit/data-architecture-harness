---
trigger: glob
globs: ["tests/**", "**/*_test.*", "**/test_*.py", "**/schema.yml", "**/schema.yaml"]
description: Testing and data quality requirements
---

# Testing & Data Quality

## The testing pyramid for data

```
        ┌───────────────────┐
        │   Contract tests  │   ← block on PR; catch breaking changes
        ├───────────────────┤
        │ Integration tests │   ← run on staging; catch wiring errors
        ├───────────────────┤
        │   Unit tests      │   ← run on every commit; catch logic errors
        ├───────────────────┤
        │   Data quality    │   ← run on every refresh; catch data errors
        └───────────────────┘
```

## Mandatory tests per model

Every model in `marts/` ships with:

1. **Grain tests** — `unique` + `not_null` on the primary key column(s).
2. **Referential tests** — `relationships` on every foreign key.
3. **Range tests** — `accepted_values` for low-cardinality dimensions;
   numeric ranges for measures.
4. **Freshness tests** — at the source level, enforced.
5. **Volume tests** — row count within a band, e.g. ±25% of the 7-day moving
   average for critical facts.

## When to write a custom test

A custom test (singular SQL test or generic test) is required when a business
invariant cannot be expressed as a built-in:

- "Sum of line revenue equals header total per order."
- "No customer has overlapping active subscriptions."
- "Refunds never exceed the original transaction amount."

Each such test has a name that reads as a sentence, e.g.
`assert_line_revenue_sums_to_order_total`.

## Severity

- `error` — block the pipeline. Use for grain, referential, and contract
  violations.
- `warn` — log and continue. Use for soft thresholds (volume anomalies, drift).
- Never use `warn` for grain or PII tests. Those are always `error`.

## Unit testing transformations

Logic in SQL or Python transformations is unit-tested with fixed input/output
fixtures. The fixture lives next to the test, in YAML or CSV. The test runs
in CI on every PR.

Examples of "logic worth unit-testing":

- SCD2 valid_from/valid_to assignment.
- Currency conversion with date-aligned rates.
- De-duplication rules with tie-breakers.
- Window functions whose ordering matters.

## CI gates

A PR cannot merge unless:

- All `error`-severity tests on changed models pass against the staging build.
- The contract on any modified mart still validates.
- A lineage diff is attached to the PR.

## What `done` looks like for a data quality issue

- [ ] Root cause identified — *upstream*, not just at the symptom.
- [ ] Test added that would have caught the issue at the right layer.
- [ ] Runbook updated if the fix involves operator action.
- [ ] Affected downstream consumers notified if their numbers shifted.
