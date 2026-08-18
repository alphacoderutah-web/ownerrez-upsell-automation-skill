---
name: ownerrez-upsell-automation
description: Build a safe, auditable automation worker that qualifies guest-approved paid add-ons (extra nights, early check-in, late check-out, or similar upsells) from OwnerRez Rezzy tasks and guest messages, applies the charge and payment through OwnerRez, and notifies staff using OwnerRez's own native Triggers and Templates instead of a separate SMS vendor. Use when asked to automate upsells or paid add-ons for an OwnerRez-managed vacation rental business, to safely charge a guest-approved fee through OwnerRez, or to set up staff SMS/email alerts tied to booking events (check-in/out time changes, new calendar blocks, payments) without standing up a separate messaging platform.
---

# OwnerRez upsell automation

This automates real money moving on a real guest's card, using a system with real gaps between what its API can do and what only its web UI can do. Every fact in this skill was learned by actually hitting the failure it describes, not by reading OwnerRez's own docs — treat that as a reason to trust the specifics and still verify them against the live account you're working in, since OwnerRez ships changes to its own UI over time.

## Non-negotiables

1. **Dry-run by default, always.** The worker should compute and log exactly what it would do, and never mutate anything until an operator has deliberately turned on live mode. The section below spells out what "deliberately" needs to mean in practice.
2. **Never invent a fee.** Real fee amounts come from the business owner, full stop. When no approved rule exists for a property/service pair, the correct behavior is to route that case to a human, not to guess a plausible number from a similar property.
3. **Never trust a UI success screen as proof.** OwnerRez's browser UI redirecting to a success-looking page, or showing a success toast, is not evidence a mutation actually landed correctly. Re-read the record fresh, or check a documented read endpoint, before treating any write as real. This single habit will save you more debugging time than anything else in this skill.
4. **Verify every contact before you send anything to it.** A staff member's phone number must come from a canonical source — a working Template's "To" field, a Team/Staff record, or the person directly — never inferred from message history. Watch specifically for guest/staff name collisions: a guest can coincidentally share a name with a staff member, and a business's own public contact number can appear attached to an unrelated guest's booking. Flag any collision you find to the business owner explicitly rather than silently picking one.

## What OwnerRez's API can and can't do

Three operations have no documented API and require driving the actual OwnerRez web UI, full stop:

1. **Reading, filtering, or resolving Rezzy tasks.** No task resource exists anywhere in the documented v2 API.
2. **Adding a charge line to an existing booking.** The booking write model has no charges field — it only exposes things like arrival, departure, check-in/out time, guest, notes, property, title.
3. **Submitting a card-on-file payment.** The payments endpoint is read-only.

Everything else should go through the documented v2 API: reading and creating bookings, checking availability, creating calendar blocks (a block is just a booking with `is_block: true`), updating check-in/out times, and — this one matters — verifying that a payment actually landed, via the documented payments-read endpoint. Never trust a UI success toast for a payment; always verify against that endpoint before treating a charge as real money collected.

Read `references/capability-matrix.md` for the full list of UI quirks that will otherwise cost you real time: tax lines that do not recalculate, a payment-scheduling checkbox that defaults to the wrong state, Airbnb's on-the-hour rounding, how to actually confirm a card is on file, and how to handle OTA-sourced bookings.

## Investigate before writing anything

Do not start from the capability matrix above as gospel for a specific account — start from it as a strong prior, then verify against the live account you are actually building for. OwnerRez accounts vary in configuration, and the fastest way to build the wrong thing is to assume this account works exactly like the last one.

Concretely: map which of the operations you need are API-reachable versus browser-only for this specific worker, get every fee amount confirmed by the business owner in writing (a chat message is fine, a guess is not), and confirm every staff contact against a canonical source per non-negotiable 4 above. If you find yourself about to type in a phone number or a dollar amount that nobody explicitly gave you, stop — that is exactly the moment this skill exists to catch.

## Safety architecture

This is the pattern to replicate in whatever language or framework the worker is built in — none of it is tied to a specific stack.

**Dry-run by default**, with live mutations gated behind several independent conditions that must ALL be true at once: an explicit live-mode config flag, an explicit runtime or CLI "execute" flag separate from that config flag, a narrow allowlist of specific bookings or properties (never "all bookings" as a default), and a hard ceiling on charge amount. Any one of these being missing or false should silently downgrade to dry-run and log why — never error out, and never fall through to live behavior by default. The value of stacking several independent conditions instead of one is that a single mistake, like a stale config flag left on or a bad copy-paste, cannot alone put real money in motion.

**Authorization requires an unambiguous chain**, not an inference: a host message quoting an explicit price, followed by an unambiguous affirmative from the guest specifically — not staff, and not the AI messaging assistant's own paraphrase of what it thinks the guest meant. A bare "yes" or "ok" with no transactional commitment attached is not sufficient by default; require something that clearly signals "charge me," like "yes please," "go ahead," or "book it." A later guest message that cancels or changes the request must always override a prior acceptance and route to a human — never auto-process or auto-refund based on your own read of an ambiguous follow-up.

**Idempotent, resumable state machine.** Every mutation step should be safely re-checkable: if the process crashes or a booking gets reprocessed, re-check what already happened via the documented read endpoints before mutating again — has this charge already been added, has this payment already landed — rather than blindly repeating a write. Never auto-retry an ambiguous payment outcome; reconcile against the read endpoint first and require a human decision before ever resubmitting a charge whose outcome is not certain.

**No automatic rollback.** If a mutation succeeds and a later step in the same workflow fails, preserve the state, flag it for human review, and alert — do not attempt to programmatically undo a real charge or booking change. Undoing money movement is itself a risky action and belongs to a human.

**Audit trail and redaction.** Log every decision and mutation with enough detail to reconstruct it later, but redact card numbers, tokens, and full guest message bodies from anything written to a log.

## Native staff alerts via OwnerRez Triggers and Templates

OwnerRez has its own built-in automation system for exactly this — Settings > Templates and Settings > Triggers — and it means staff-facing SMS and email alerts do not need a separate vendor, Twilio or otherwise, integrated and maintained by your worker at all. This is the single most valuable, least-documented thing this skill knows, and it comes with three gotchas that will each independently produce a save that looks successful while silently missing the field that actually matters.

Read `references/triggers-and-templates.md` before building any trigger-based notification. Do not try to reconstruct this from OwnerRez's UI alone — the failure modes here look exactly like success unless you specifically know to check for them.

## Pre-launch checklist

Before any of this touches a real guest's card, work through `references/safety-checklist.md` — a canary-style rollout: one booking, narrow allowlist, verify in OwnerRez itself rather than your own logs, then widen deliberately.

## Reference files

Read these when the situation calls for them rather than upfront.

- **`references/capability-matrix.md`** — the full API-vs-UI breakdown and every UI quirk that costs real time: tax-line recalculation, the Schedule Payments checkbox, Airbnb's on-the-hour rule, how to actually confirm a card is on file, OTA handling, and rate limits.
- **`references/triggers-and-templates.md`** — the full Triggers + Templates playbook: the event/action/condition model, how template types and merge fields work, how to map an upsell type to the right event without needing content-matching (which OwnerRez does not support), and the three gotchas that will silently eat a trigger's configuration if you do not know about them.
- **`references/safety-checklist.md`** — a terse, actionable pre-launch checklist for taking a worker from dry-run to a single verified live canary to a wider rollout.
