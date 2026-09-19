---
id: native-triggers-for-staff-alerts
title: "Staff alerts use OwnerRez's native Triggers + Templates, not a separate SMS vendor"
category: decision
status: active
tags: [staff-alerts, triggers, templates]
created: "2026-09-18T18:27:03"
updated: "2026-09-18T18:27:34"
---

<!-- compiled_truth -->
Staff SMS and email alerts come from OwnerRez Triggers + Templates fired by the worker's natural booking or block event; three UI gotchas can silently empty the Action.

## What was decided

Staff-facing SMS and email alerts are configured in OwnerRez itself (Settings > Templates and Settings > Triggers) instead of integrating and maintaining a separate SMS vendor inside the worker. OwnerRez sends from its own configured number as part of the account's existing subscription. The skill calls this its most valuable and least-documented piece of knowledge.

## The model

- A **Trigger** pairs an **Event** with an **Action**. Events are immediate (for bookings: created, canceled, dates changed, check-in/out time changed, notes changed, payment added; for blocked-off time: created, updated, deleted) or scheduled (a number of days or hours before or after arrival, departure or booking creation).
- The Action is a saved, **typed Template** (Email, SMS or Channel Message). A trigger only offers templates whose type matches its event category, and each type has its own merge fields. A Booking template can show both the previous and the new check-in/out time; a Blocked-Off Time template has only the block's own fields.
- **Conditions** are structured fields only: properties (a real multi-select), dates, statuses, tags, guest counts and similar. There is no free-text or content-matching condition.

## How an upsell maps to an event

Content cannot be matched, so the trigger fires on the event the worker's own mutation naturally produces:

- a time-based upsell (early check-in, late check-out) changes the booking's check-in/out time, so it uses the time-changed event;
- a night-based upsell implemented as a calendar block uses the blocked-off-time-created event.

Firing on a generic event such as "payment added" and filtering by amount is ruled out: there is no field to filter on, so it would alert on every payment. To tell automation apart from a manual staff edit, the supported mechanism is a **Tag condition** (the worker tags the booking), at the cost of an extra write with an ordering dependency. The skill's default is to fire on the natural event unconditionally, because staff usually want to hear about any time change or new block.

## The three save gotchas

Each one produces a trigger that saves and redirects like a success while its Action is empty or wrong:

1. **Hidden radio inputs.** The Action dropdown's real state lives in a hidden set of radio inputs, not in the visible select, so setting the select programmatically is overwritten at save. Use the visible widget the way a person would, or set the matching radio and dispatch its change and click events, then re-check the select.
2. **Asynchronous reload.** Changing the Event reloads the Action's valid options and wipes an Action chosen too early. Set the Event first, let it settle, and set the Action last, right before saving.
3. **The save response proves nothing.** Load the record's edit URL fresh and re-read it; see [[ownerrez-state-is-source-of-truth]]. The list page's running total is a cheap check that a batch landed with no duplicates or orphans.

## Inputs rule

Every recipient comes from a canonical source: a working Template's To field, a Team/Staff record, or the person directly. Never infer one from message history. A guest who shares a name with a staff member, or a business's own public number appearing on an unrelated guest's booking, is flagged to the business owner, not resolved silently.

## Trade-offs and boundary

- Accepted: alerts fire on any matching event, including manual edits.
- Out of scope: guest-facing triggers such as a pre-arrival email. It is the same mechanism for a different purpose, and a deliberate should-not-trigger case in [[trigger-description-evals]].


## Timeline

- time: 2026-09-18T18:27:03
  kind: decision
  summary: "Created this page: Staff alerts use OwnerRez's native Triggers + Templates, not a separate SMS vendor"
  source: "SKILL.md; references/triggers-and-templates.md; commit 0bbc946"
  affects: [native-triggers-for-staff-alerts]

- time: 2026-09-18T18:27:03
  kind: decision
  summary: "captured from project history: native alert model, event mapping, three save gotchas, contact provenance"
  source: references/triggers-and-templates.md
  affects: [native-triggers-for-staff-alerts]

- time: 2026-09-18T18:27:34
  kind: decision
  summary: "lead with a one-line summary so the index entry is informative; content unchanged"
  source: same as previous entry
  affects: [native-triggers-for-staff-alerts]
