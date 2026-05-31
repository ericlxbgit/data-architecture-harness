# Windsurf Harness — SQL Server 3NF Data Modeling

A focused two-file rule set that configures Windsurf's Cascade agent to act
as a Principal Data Architect on SQL Server, enforcing Third Normal Form
discipline.

## What's in here

```
.windsurf/
└── rules/
    ├── 10-data-modeling-3nf.md      Modeling discipline (always on)
    └── 20-sqlserver-conventions.md  SQL Server naming, types, DDL (always on)
```

Both files use `trigger: always_on`, so the agent has them loaded on every
turn. That is appropriate here because the scope is narrow — when the whole
job is "model this in 3NF on SQL Server," every interaction needs both rule
sets.

## Setup

1. Drop `.windsurf/` into the root of your repo.
2. Commit it. Anyone on the team who clones the repo and opens it in
   Windsurf gets the rules automatically.
3. Smoke test: ask Cascade *"What primary key type should I use for a new
   Customer table?"* — you should get `INT IDENTITY` clustered, named PK,
   with a `UNIQUE` constraint on the natural key. If you do, the rules are
   loading correctly.

## Day-to-day use

Just ask the agent to model things. It will:

- Walk the 3NF process (entity → natural key → attributes → dependency
  check → relationships → constraints → DDL).
- Refuse to denormalize without a stated reason.
- Refuse common anti-patterns (EAV, magic values, deprecated types).
- Apply SQL Server-specific conventions (PascalCase, singular table names,
  `NVARCHAR`, `DATETIME2`, explicit constraint names, FK indexes).

When it makes a non-obvious call, ask it to cite the rule — that's both an
audit trail and a teaching moment.

## Extending the harness

If your project grows beyond pure modeling — pipelines, security, testing,
migrations — add new rule files alongside these two. Use the numeric prefix
to control reading order: `30-*.md`, `40-*.md`, etc. Each file's
front-matter (`trigger:` and `description:`) controls when it loads.

For specialized topics that don't apply to every request (e.g. PII
handling, anti-patterns reference), prefer `trigger: model_decision` with a
clear `description:` — that lets Cascade pull the file in only when
relevant, preserving context budget.

## Promoting across projects

The cheapest path: copy this `.windsurf/` folder into each new SQL Server
project. For more than 3-4 projects, create a GitHub template repository
with this folder pre-installed, and start new projects from it.

Changes to the rules should go through PR review like any other code — the
rules are the codified, version-controlled team consensus, not someone's
personal preference.
