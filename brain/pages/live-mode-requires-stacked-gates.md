---
id: live-mode-requires-stacked-gates
title: "Dry-run by default; live mutations need four independent gates at once"
category: decision
status: active
tags: [safety, dry-run, rollout]
created: "2026-09-18T18:27:03"
updated: "2026-09-18T18:27:34"
---

<!-- compiled_truth -->
Dry-run is the default; live mutation needs a config flag, a separate execute flag, a narrow allowlist and a charge ceiling, all at once.

## What was decided

The worker runs in dry-run by default: it computes and logs exactly what it would do and mutates nothing. Live mutation happens only when **all** of these are true at the same moment:

1. an explicit live-mode setting in configuration;
2. a separate, explicit execute flag at run time (CLI or runtime), distinct from that setting;
3. a narrow allowlist of specific bookings or properties, never "all bookings" as a default;
4. a hard ceiling on the charge amount.

If any one is missing or false, the worker quietly downgrades to dry-run and logs why. It does not error out on that, and it never falls through to live behaviour by default.

## Alternatives considered

- **One live flag.** Rejected: a single stale flag or bad copy-paste would be enough to move real money.
- **Failing loudly when a gate is missing.** Not chosen; the prescribed behaviour is a logged downgrade to dry-run.

## Rationale

A wrong charge cannot be quietly undone, so the design makes any single mistake insufficient to put money in motion. Once live, the allowlist and the ceiling also bound the blast radius of a bug.

## How rollout uses the gates

The canary checklist narrows the gates on purpose: the allowlist holds exactly one booking, and the ceiling sits just above the one expected amount rather than at a generous round number, because its job is to bound a bug, not to fit every future charge. Only after one clean, fully verified canary does the allowlist widen, in separate steps: one booking, then a property, then the whole account.

## Blast radius

Every worker built with the skill, and the pre-launch checklist. No amounts are recorded here; ceilings are set per account from owner-approved fees (see [[charge-authorization-chain]]).

Related: [[ownerrez-state-is-source-of-truth]].


## Timeline

- time: 2026-09-18T18:27:03
  kind: decision
  summary: "Created this page: Dry-run by default; live mutations need four independent gates at once"
  source: "SKILL.md safety architecture; references/safety-checklist.md; commit 0bbc946"
  affects: [live-mode-requires-stacked-gates]

- time: 2026-09-18T18:27:03
  kind: decision
  summary: "captured from project history: multi-gate live mode and how the canary narrows it"
  source: "SKILL.md; references/safety-checklist.md"
  affects: [live-mode-requires-stacked-gates]

- time: 2026-09-18T18:27:34
  kind: decision
  summary: "lead with a one-line summary so the index entry is informative; content unchanged"
  source: same as previous entry
  affects: [live-mode-requires-stacked-gates]
