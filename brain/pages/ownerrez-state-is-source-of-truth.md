---
id: ownerrez-state-is-source-of-truth
title: "OwnerRez's fresh re-read, not a success screen or the worker's log, proves a mutation"
category: concept
status: active
tags: [safety, verification, idempotency]
created: "2026-09-18T18:27:03"
updated: "2026-09-18T18:27:34"
---

<!-- compiled_truth -->
A write is done only when a fresh OwnerRez read shows it; this drives idempotent steps, no blind retry, no auto-rollback and a redacted audit log.

## Definition

A mutation counts as done only when OwnerRez itself shows it on a fresh read: a documented read endpoint, or the record's own edit page loaded anew. A success toast, a redirect to a success-looking URL, or the worker's own log line is not evidence.

## Where it applies

- **Payments.** After Collect Payment in the UI, confirm through the documented payments-read endpoint before treating the charge as money collected.
- **Charge lines.** Confirm the booking balance changed by exactly the intended amount before moving on to payment, and stop if it did not (an appended charge may sit outside a percentage tax line).
- **UI controls.** Re-read a control's actual state, such as the "Schedule Payments?" checkbox, instead of trusting that a click landed.
- **Triggers and Templates.** After saving, load the record's edit URL fresh and re-read the Action field; see [[native-triggers-for-staff-alerts]].
- **The canary.** Success is judged in OwnerRez: exactly one new charge line, one payment, the correct booking state, and the staff alert actually sent (the Trigger's send history, or the recipient). Not in the worker's logs.

## What follows from it

- **Idempotent, resumable steps.** Before each mutation, check through the read endpoints whether it already happened (charge already added? payment already landed?), so a crash or a reprocessed booking never repeats a write blindly.
- **No blind retry.** An ambiguous payment outcome is reconciled against the payments-read endpoint and then needs a human decision before any resubmission.
- **No automatic rollback.** If one step succeeds and a later one fails, preserve the state, flag it for review and alert. Undoing money movement is itself a risky action and belongs to a human.
- **Audit trail with redaction.** Log every decision and mutation in enough detail to reconstruct it later, but keep card numbers, payment tokens and full guest message bodies out of logs.

## Why it is this way

OwnerRez's UI can report success while the one field that mattered silently failed to persist. The skill names this habit as the single biggest saver of debugging time it offers, and applies it to every UI mutation, not only Triggers.

## Boundaries

- It governs proof of writes. It does not say where qualification data is read from.
- The worker's log still matters as the audit record; it is just not proof that OwnerRez changed.

Related: [[api-first-browser-only-for-gaps]], [[live-mode-requires-stacked-gates]].


## Timeline

- time: 2026-09-18T18:27:03
  kind: decision
  summary: "Created this page: OwnerRez's fresh re-read, not a success screen or the worker's log, proves a mutation"
  source: "SKILL.md; references/triggers-and-templates.md gotcha 3; references/safety-checklist.md"
  affects: [ownerrez-state-is-source-of-truth]

- time: 2026-09-18T18:27:03
  kind: decision
  summary: "captured from project history: verify-by-re-read, idempotency, no blind retry, no auto-rollback, redacted audit"
  source: "SKILL.md; references/*.md"
  affects: [ownerrez-state-is-source-of-truth]

- time: 2026-09-18T18:27:34
  kind: decision
  summary: "lead with a one-line summary so the index entry is informative; content unchanged"
  source: same as previous entry
  affects: [ownerrez-state-is-source-of-truth]
