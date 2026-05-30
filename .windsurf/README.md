# Windsurf Harness — Data Architecture

This directory configures **Windsurf's Cascade agent** to act as a Principal
Data Architect on this repo. It enforces our principles, naming conventions,
testing requirements, and security rules consistently across every developer
on the team.

If you have Windsurf open in this repo, the agent has already loaded these
rules. You don't have to do anything to "turn it on."

---

## What's in here

```
.windsurf/
├── rules/                          ← always- or context-loaded instructions
│   ├── 00-role-and-soul.md         Identity & communication style (always on)
│   ├── 10-principles.md            Core principles (always on)
│   ├── 20-policies-guardrails.md   Hard rules / refusals (always on)
│   ├── 30-naming-and-structure.md  Conventions (always on)
│   ├── 40-tech-stack.md            Our stack — FILL IN per project (always on)
│   ├── 50-data-modeling.md         Modeling rules (glob: models/**)
│   ├── 60-pipelines.md             Orchestration rules (glob: pipelines/**)
│   ├── 70-security-pii.md          PII handling (model decision)
│   ├── 80-testing-quality.md       Test requirements (glob: tests/**)
│   └── 99-antipatterns.md          What to avoid (model decision)
└── workflows/                      ← reusable procedures, invoked with /name
    ├── new-pipeline.md             /new-pipeline
    ├── new-data-model.md           /new-data-model
    ├── schema-migration.md         /schema-migration
    └── architecture-review.md      /architecture-review
```

## How rules activate

| Mode             | When it loads                                        | Used for                          |
|------------------|------------------------------------------------------|-----------------------------------|
| `always_on`      | Every Cascade turn                                   | Identity, principles, policies    |
| `glob`           | When files matching the glob are open or edited      | Layer-specific conventions        |
| `model_decision` | When Cascade judges the rule is relevant             | Specialized topics (PII, anti-patterns) |
| `manual`         | Only when you explicitly @ the rule                  | Rare, one-off procedures          |

Keeping rules out of `always_on` matters: rule text consumes context budget.
Loading PII or anti-pattern rules only when they're needed gives the agent
more room to think about your actual problem.

---

## How to use it day-to-day

### Starting a new piece of work

Pick the matching workflow and type the slash command in Cascade:

```
/new-pipeline      — full source-to-mart pipeline
/new-data-model    — a single model in an existing pipeline
/schema-migration  — changing an existing table safely
/architecture-review — reviewing someone else's design
```

Cascade will walk the workflow, asking the right questions in the right order.

### Asking ad-hoc questions

Just ask. The role, principles, policies, and conventions are always loaded,
so the agent will already reason from them. If it makes a non-obvious call,
ask it to cite which rule it invoked — that's a teaching opportunity for both
you and the rule set.

### When the agent refuses

The agent will refuse requests that violate `20-policies-guardrails.md`. This
is intentional. If you believe the refusal is wrong:

1. Read the cited policy.
2. If the policy itself is wrong for this case, open a PR to change the
   policy — don't argue with the agent.
3. If your request is fine and the policy is fine but the agent
   misclassified it, rephrase with more context.

---

## Setting up Windsurf on this repo (for new team members)

1. Install Windsurf: <https://windsurf.com/download>.
2. Clone this repo and open it in Windsurf.
3. Open Cascade (the AI panel). The rules in `.windsurf/rules/` load
   automatically — no settings to flip.
4. Verify by asking Cascade: *"What's our policy on renaming columns?"*
   It should cite `20-policies-guardrails.md` and `50-data-modeling.md` and
   describe the expand–migrate–contract pattern.
5. Read `40-tech-stack.md` to learn the project's specific stack.

### Personal global rules (optional)

Your *personal* preferences — preferred SQL style for your own scratch code,
your favorite test framework, how you like to be addressed — go in your
**global** rules, not here. Set them in Windsurf: **Settings → Cascade →
Custom Instructions** or in `~/.codeium/windsurf/memories/global_rules.md`.

Workspace rules (this directory) always take precedence over global rules
when they conflict.

---

## How to evolve the harness

These files are not sacred. They are the codified, version-controlled
distillation of the team's current thinking. They should change as we learn.

### To propose a change

1. Open a PR that edits the relevant file in `.windsurf/`.
2. In the PR description, explain *why*. Reference the case or incident
   that prompted the change.
3. Tag at least two members of the data team for review.
4. Once merged, post in the data channel: *"FYI: updated
   `20-policies-guardrails.md` to require X. Cascade now enforces this in
   every project that pulls this template."*

### What goes in rules vs workflows

- **Rules** are *constraints*: "always do X" / "never do Y" / "name things
  this way." They are loaded automatically and shape every interaction.
- **Workflows** are *procedures*: ordered steps for a specific task. They
  are invoked deliberately with a slash command.

If you find yourself writing "first do A, then do B, then do C" in a rule
file, it probably belongs in a workflow.

### What goes in `always_on` vs other triggers

- **Always-on**: identity, top principles, hard policies, repo-wide
  conventions. If it's universal, it's `always_on`.
- **Glob**: layer-specific rules that only matter when you're in that layer.
- **Model decision**: specialized topics (PII, anti-patterns, compliance)
  that aren't relevant to every request.

Don't put everything in `always_on`. Context budget is real.

---

## Promoting this harness across projects

Three options, in order of investment:

### Option 1 — Copy/paste (lowest effort)

For each new project, copy `.windsurf/` from this repo into the new repo.
Edit `40-tech-stack.md` for the new project's stack. Commit.

### Option 2 — Template repository (recommended)

Create a `data-platform-template` repo containing a skeleton with this
`.windsurf/` directory pre-installed, along with whatever else you start
new data projects with (Makefile, CI config, dbt project skeleton, IaC
skeleton). Use GitHub's "Template repository" feature so anyone can spin
up a new project from it with one click.

When you update a rule in the template, document the change and notify
existing projects so they can pull the diff if they want it.

### Option 3 — Centralized rules with per-project overlays (highest effort)

Maintain a single source-of-truth repo for the rules. Each project pulls
them in via a git submodule or a CI sync job, and keeps its own
`40-tech-stack.md` and any project-specific rules locally. Changes to the
central rules propagate automatically.

This is worth doing only once you have 5+ active data projects. Below that,
a template repo is simpler and almost as good.

---

## FAQ for the team

**Q: Cascade ignored my rule. What happened?**
Three usual causes: (1) the rule's activation mode didn't match the
context (check `trigger:` and `globs:` in the front-matter); (2) another
rule contradicts it and won — make the priority explicit in the text;
(3) the rule was too vague — be specific and use examples.

**Q: Can I bypass the rules for a one-off?**
Yes — just tell Cascade explicitly in your message, with a reason. The
agent will note the deviation. Don't edit rules for one-offs.

**Q: The agent keeps citing a rule I don't agree with.**
Open a PR to change the rule. That is the supported feedback channel. If
you bypass it locally, the rest of the team still sees the old behavior.

**Q: Do these rules apply outside Cascade (e.g. when I write code by hand)?**
Yes — they encode the team's standards. Cascade just enforces them
automatically. They are equally binding on humans.
