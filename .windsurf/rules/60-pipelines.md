---
trigger: glob
globs: ["pipelines/**", "dags/**", "flows/**", "**/*_dag.py", "**/*_flow.py"]
description: Pipeline and orchestration patterns
---

# Pipelines & Orchestration

## Core requirements

Every pipeline you write or modify must satisfy these or you do not merge:

1. **Idempotent**: re-running with the same partition / window produces the
   same end state. Use upserts or partition-replace, never blind inserts.
2. **Restartable**: a failed run can be re-run from the failed task without
   re-doing successful upstream work.
3. **Observable**: emits structured logs, task duration, row counts, and a
   success/failure metric to the monitoring stack.
4. **Bounded**: every task has a timeout and a retry policy with backoff.
5. **Atomic publish**: consumers see the new partition or the old one, never
   a half-written state. Use staging tables + swap, or transactional writes.

## Retry policy defaults

- Transient failures (network, rate limit): 3 retries, exponential backoff
  starting at 60s.
- Data quality failures: 0 retries. Fail loud — retrying bad data is worse
  than no data.
- Resource exhaustion: 1 retry after a longer delay, then page.

## Dependencies

- Express data dependencies as **data-aware**, not time-aware, wherever the
  orchestrator supports it (Airflow datasets, Dagster assets, etc.).
- Time-based triggers are only acceptable for true upstream entry points
  (e.g. "poll the API at 02:00").
- Cross-DAG dependencies use sensors with timeouts, not bare schedule
  alignment. Schedule drift is real.

## Configuration

- No hardcoded environment names, project IDs, dataset names, paths, or URLs.
  Everything comes from config keyed by environment.
- Secrets come from the secret manager. Never from env vars baked into
  container images. Never from config files in the repo.

## Backfills

- Every pipeline has a documented backfill procedure in its docstring or
  adjacent runbook: how to invoke, expected duration, cost estimate, locks
  taken.
- Backfills run with the same code as forward runs. No special "backfill
  mode" forks of the logic.
- A backfill that would overwrite more than 30 days of partitions requires
  written sign-off — the agent should refuse without it.

## Failure handling

- Pipelines fail closed. A downstream mart blocks rather than serving stale.
- Alerts route to the team owning the *failing task*, not a generic channel.
- Every failure produces a structured log with: pipeline name, task name,
  partition / window, error class, and a link to the run.

## What "done" means for a pipeline change

- [ ] Code change has tests (unit for transforms, integration for the DAG).
- [ ] Runbook updated if behavior changed.
- [ ] Monitoring updated (new tasks have alerts; removed tasks don't page).
- [ ] Backfill plan documented if schema changed.
- [ ] PR description explains the *why*, not just the *what*.
