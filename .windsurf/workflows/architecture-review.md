---
description: Review a proposed data architecture or design against the project's principles
---

# Workflow: Architecture Review

Use when reviewing a proposed design — a new mart, a new pipeline pattern,
a refactor, or a doc/ADR — against this project's principles.

## Step 1 — Restate the proposal

In your own words, summarize:

- What is being built or changed.
- What problem it solves.
- Who the consumers are.
- What the proposer claims as the trade-offs.

If you can't restate it without ambiguity, ask before reviewing.

## Step 2 — Check against the principles

Walk `10-principles.md` top to bottom. For each principle, write one line:

- **Pass** — design respects this principle.
- **Concern** — design tensions with this principle; flag and explain.
- **Fail** — design violates this principle; this must change before approval.

Do not skip principles you "trust" the proposer on. Walk all twelve.

## Step 3 — Check against the policies

Walk `20-policies-guardrails.md`. Any policy violation is a blocker. Cite by
name.

## Step 4 — Look for anti-patterns

Cross-reference against `99-antipatterns.md`. If you find one, name it and
propose the documented fix.

## Step 5 — Evaluate the trade-offs the proposer named

For each trade-off they called out:

- Is the cost they described actually the largest cost? Often the real cost
  is something they didn't name.
- Is the benefit measurable? If not, ask how they'd know they were wrong.

## Step 6 — Evaluate the trade-offs the proposer *didn't* name

These are the most common omissions in data architecture proposals:

- Cost at full data volume (not just the prototype).
- Operability — who pages when this breaks at 03:00?
- Lineage impact — does this widen or narrow the blast radius of upstream
  changes?
- Reversibility — if we ship this and regret it, how do we undo?
- PII propagation — does this change which roles can see which data?

For each omission, ask a specific question.

## Step 7 — Recommendation

End with one of:

- **Approve** — ship as proposed.
- **Approve with non-blocking suggestions** — ship after the listed nits.
- **Approve with required changes** — ship after specific changes; re-review.
- **Reject** — fundamental issues; propose a different approach.

Use these words exactly. Don't soften the recommendation.

## Tone

This is a peer review, not a takedown. Lead with what the proposal does
*well*, then the concerns. Concerns are about the design, never the
proposer.
