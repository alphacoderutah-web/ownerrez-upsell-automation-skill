---
id: charge-authorization-chain
title: "A charge needs a quoted price, the guest's own unambiguous yes, and an owner-approved fee rule"
category: decision
status: active
tags: [safety, authorization, fees]
created: "2026-09-18T18:27:03"
updated: "2026-09-18T18:27:34"
---

<!-- compiled_truth -->
Authorized only with a host-quoted price, the guest's own clear charge-me reply and an owner-approved fee rule; anything less goes to a human.

## What was decided

A worker may treat an upsell as authorized only when the whole chain is present:

1. **A host message quoting an explicit price** for the add-on.
2. **An unambiguous affirmative from the guest themself** — not from staff, and not the messaging assistant's paraphrase of what it thinks the guest meant. A bare "yes" or "ok" with no transactional commitment is not enough by default; the reply has to clearly mean "charge me", as "go ahead" or "book it" do.
3. **An owner-approved fee rule** for that exact property and service pair, confirmed in writing by the business owner. A chat message counts as writing; a guess does not.

Two overrides always apply:

- A later guest message that cancels or changes the request beats an earlier acceptance and routes to a human. The worker never auto-processes or auto-refunds on its own reading of an ambiguous follow-up.
- No approved rule for a property/service pair means human review. The worker never invents or estimates a fee, and never borrows one from a similar-looking property.

## Alternatives considered

- **Inferring consent from a short reply or from context.** Rejected: inference is exactly where a guest who never agreed gets charged.
- **Filling a missing fee from a comparable property.** Rejected for the same reason: the number has to come from the owner.

## Rationale

Charging someone who did not agree, or charging the wrong amount, is the failure the skill exists to prevent. Requiring explicit evidence at every link, and turning anything short of it into a human decision, removes the guesswork. The skill frames the moment someone is about to type an amount nobody explicitly gave them as the exact moment it exists to catch.

## Mechanism, not figures

Neither the skill nor this brain holds fee amounts. Quoted prices come from the conversation, approved amounts from the owner, and the live-mode ceiling bounds any single charge; see [[live-mode-requires-stacked-gates]].

## Blast radius

The qualification logic of every worker, and the first item of the pre-launch checklist (every live fee confirmed in writing for each property/service pair).


## Timeline

- time: 2026-09-18T18:27:03
  kind: decision
  summary: "Created this page: A charge needs a quoted price, the guest's own unambiguous yes, and an owner-approved fee rule"
  source: "SKILL.md non-negotiables and safety architecture; commit 0bbc946"
  affects: [charge-authorization-chain]

- time: 2026-09-18T18:27:03
  kind: decision
  summary: "captured from project history: authorization chain, override rules, no invented fees"
  source: "SKILL.md; references/safety-checklist.md"
  affects: [charge-authorization-chain]

- time: 2026-09-18T18:27:34
  kind: decision
  summary: "lead with a one-line summary so the index entry is informative; content unchanged"
  source: same as previous entry
  affects: [charge-authorization-chain]
