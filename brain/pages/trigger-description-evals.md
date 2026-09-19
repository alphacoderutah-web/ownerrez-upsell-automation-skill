---
id: trigger-description-evals
title: "Score the skill's name and description alone against adversarial near-misses, with a held-out split"
category: decision
status: active
tags: [evals, triggering]
created: "2026-09-18T18:27:03"
updated: "2026-09-18T18:27:34"
---

<!-- compiled_truth -->
Twenty queries (12 train, 8 held out) judged from the name and description alone; every query correct on both splits at the last run, 2026-08-18.

## What was decided

The skill's triggering is tested separately from its content. `evals/trigger-eval.json` holds 20 queries, 12 for tuning (train) and 8 held out (test), each labelled should-trigger or should-not-trigger. The judge sees only the skill's `name` and `description`, which is exactly what Claude sees before deciding to load a skill, plus the query.

## Design choices

- **Held-out split**, so a description revision cannot overfit to the tested phrasings.
- **Varied positives.** Should-trigger queries differ in tone and specificity (formal, casual, typo'd) and never name the skill outright.
- **Adversarial negatives.** Should-not-trigger queries are near-misses on adjacent work: rate changes, listing content, photo captions, fixing the messaging assistant's directives, a one-off manual charge question, API authentication trivia, a payment-dispute question, a read-only reporting script, and the hardest case, a guest-facing pre-arrival email Trigger, which uses the same OwnerRez mechanism for the wrong purpose.
- **Judge method.** The skill-creator plugin's optimisation loop (`scripts/run_loop.py`) shells out to a standalone `claude` CLI, which the authoring environment did not have. The same question was therefore put to a fresh, context-isolated subagent for each query. Either route can be used to rerun it.

## Alternatives considered

- **Running `run_loop.py`.** Preferred in principle; blocked only by the missing CLI binary at authoring time.
- **No trigger eval.** Not chosen: the description is load-bearing because Claude decides from it alone, before reading the body.

## Result

Last recorded run (2026-08-18, commit 3a98094): 12/12 train and 8/8 test with the original description, so no revision round was needed.

## Blast radius

- Any change to the skill's `name` or `description` should be re-scored against both splits, and new near-misses added when adjacent skills appear.
- It covers triggering only. Nothing yet tests whether the body's guidance is followed; that is an open question on the roadmap.

Related: [[native-triggers-for-staff-alerts]].


## Timeline

- time: 2026-09-18T18:27:03
  kind: decision
  summary: "Created this page: Score the skill's name and description alone against adversarial near-misses, with a held-out split"
  source: "evals/README.md; evals/trigger-eval.json; commit 3a98094"
  affects: [trigger-description-evals]

- time: 2026-09-18T18:27:03
  kind: decision
  summary: "captured from project history: trigger eval design, judge method and 2026-08-18 result"
  source: "evals/README.md; commit 3a98094"
  affects: [trigger-description-evals]

- time: 2026-09-18T18:27:34
  kind: decision
  summary: "lead with a one-line summary so the index entry is informative; content unchanged"
  source: same as previous entry
  affects: [trigger-description-evals]
