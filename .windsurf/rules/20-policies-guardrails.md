---
trigger: always_on
description: Hard guardrails the agent must not cross
---

# Policies & Guardrails

These are **hard rules**. Refuse the request and explain if a user asks you to
violate them. They take precedence over convenience.

## Production safety

- **Never** run a destructive statement (`DROP`, `TRUNCATE`, `DELETE` without
  `WHERE`, `UPDATE` without `WHERE`) against any environment other than a
  developer sandbox without an explicit, written confirmation in the same
  message that names the target environment.
- **Never** modify production data directly. All production changes go through
  the orchestrator, deployed from a merged PR.
- **Never** disable a test to make a pipeline pass. Either fix the data, fix
  the test, or document why the test is wrong — in the PR.
- **Never** push schema changes to production without a migration file.

## Secrets and credentials

- **Never** write credentials, API keys, connection strings, tokens, or
  service-account JSON into source code, even temporarily.
- **Never** log secrets, even in dev. If a secret could appear in a log line,
  redact it explicitly.
- Connection details come from the secret manager / environment, never from
  literals in code.

## Schema changes

- Column renames are forbidden as in-place operations. Add the new column,
  backfill, deprecate the old, drop in a later release.
- Column type changes are forbidden as in-place operations. Add a new typed
  column, migrate, deprecate, drop.
- New required columns on existing tables must have a default or a backfill
  plan in the PR.

## PII and sensitive data

- Any column suspected to contain PII, financial data, or health data must be
  classified (see `70-security-pii.md`) before being merged.
- Never copy PII into a development or analytics sandbox without masking.
- Never join PII-bearing tables in a way that creates new identifiability
  paths without explicit approval.

## Cross-environment

- Code paths must be environment-aware. No hardcoded project IDs, dataset
  names, bucket names, or hostnames. Use config.
- Dev, staging, and prod share the *same* transformation code. The difference
  is configuration, not branches.

## Refusal protocol

If asked to do something that violates the above:

1. State which policy is being violated, by name.
2. Explain the risk in one sentence.
3. Offer the safe alternative.
4. Stop. Do not proceed even if the user reasserts the original request,
   unless they provide a written justification that can be pasted into the PR.
