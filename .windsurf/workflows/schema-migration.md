---
description: Safely change a schema (add/remove/rename columns) without breaking downstream
---

# Workflow: Schema Migration

Use this for any change to a contracted or widely-consumed table's columns,
types, or semantics.

## Step 1 — Classify the change

| Change                                  | Safe?   | Path                                  |
|-----------------------------------------|---------|---------------------------------------|
| Add nullable column                     | Safe    | Single PR.                            |
| Add non-nullable column                 | Risky   | Two-phase: backfill, then enforce.    |
| Rename column                           | Unsafe  | Expand–migrate–contract (3 PRs).      |
| Change column type (widening)           | Risky   | Two-phase: new column, migrate, drop. |
| Change column type (narrowing)          | Unsafe  | Same as rename.                       |
| Change column semantics                 | Unsafe  | New column with new name only.        |
| Remove column                           | Unsafe  | Deprecate, wait one release, drop.    |

If the change is "unsafe," do not propose a single PR. Lay out the phases.

## Step 2 — Inventory downstream consumers

Before writing any code:

1. Pull lineage for the table.
2. List every downstream model, dashboard, and service.
3. For each consumer, identify the owner.
4. Note any consumer that uses the column being changed.

Output the list to the user. Wait for confirmation before proceeding.

## Step 3 — Expand–Migrate–Contract pattern

For renames, type narrowing, semantic changes:

**PR 1 — Expand**:
- Add the new column alongside the old.
- Backfill from the old (or from a new source if semantics differ).
- Both populated in parallel from now on.
- Old column tagged `deprecated: true` in `schema.yml`.

**PR 2 — Migrate**:
- Update each downstream consumer to read the new column.
- One PR per consumer team, or grouped if owned by the same team.
- Each merge gated on tests.

**PR 3 — Contract** (after a defined waiting period, e.g. one release):
- Confirm no queries reference the old column (log audit or query history).
- Drop the old column.

## Step 4 — Communications

- Announce the migration in the data team channel **before** PR 1.
- Re-announce when entering Step 3 (drop) with a deadline.
- Each PR description links to the migration plan.

## Step 5 — Rollback plan

Every PR in the sequence is independently rollback-safe:

- PR 1 rollback: drop the new column. Old continues to work.
- PR 2 rollback: revert per consumer; both columns still present.
- PR 3 rollback: restore from the previous schema; data is already in the
  new column, so no data loss.

## Refuse if

- The user wants to skip the expand phase and rename in place. Cite policy
  `20-policies-guardrails.md`.
- The user wants to "just change the semantics, the name is fine." This is
  the most dangerous change of all — it cannot be detected by anyone reading
  the lineage. Refuse and require a new column.
