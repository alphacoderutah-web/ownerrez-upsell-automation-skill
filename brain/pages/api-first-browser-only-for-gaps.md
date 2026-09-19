---
id: api-first-browser-only-for-gaps
title: "Use the OwnerRez v2 API wherever it reaches; drive the web UI only for the three gaps"
category: decision
status: active
tags: [ownerrez-api, browser-automation, capability-map]
created: "2026-09-18T18:27:02"
updated: "2026-09-18T18:27:33"
---

<!-- compiled_truth -->
Use the documented v2 API for everything it supports; browser automation only for Rezzy tasks, adding a charge line and collecting a card payment.

## What was decided

A worker built with this skill uses OwnerRez's documented v2 API for everything the API supports, and drives the web UI through browser automation only for the three operations that have no documented API:

1. **Rezzy tasks** — reading, filtering or resolving them. No task resource exists in the documented API.
2. **Adding a charge line to an existing booking.** The booking write model has no charges field.
3. **Collecting a card-on-file payment.** The payments endpoint is read-only.

On the API side: booking reads and creates, availability checks, check-in/out time changes and other booking fields, calendar blocks (a block is a booking created with `is_block: true`, the usual way to sell an extra night without moving the guest's own dates), and payment reads. The payment read matters most, because it is how a payment collected through the UI is proven.

## Alternatives considered

- **Browser automation for everything.** Rejected: it adds fragility for operations the API already supports.
- **Treating the capability map as fixed truth.** Rejected: the skill calls it a strong prior and tells Claude to re-map API vs UI for the specific account and worker before writing anything, because accounts are configured differently and OwnerRez changes its UI.

## Rationale

The UI is where the silent failures live, so UI work is kept to the minimum. The quirks the skill records (full list in `references/capability-matrix.md`):

- A percentage tax line only covers the lines above it, and new charges are appended at the bottom, so an added charge is not taxed unless it is moved above the tax lines, or the balance change is verified to be exactly the intended amount.
- "Reset to Quote Charges" and "Reset to Property Rates" wipe every charge line on the booking.
- The "Schedule Payments?" box in Collect Payment defaults to checked and must be unchecked and re-read.
- A card charge in payment history is not a stored card: OTA bookings often show a payment the OTA processed off-platform. Check the Cards On File section.
- Airbnb only accepts on-the-hour check-in and check-out times and rounds anything else on its side.

## Blast radius

- Shapes every worker built with the skill. The two money-moving UI steps (Add Charge, Collect Payment) are the ones the canary checklist has you confirm form labels for, without saving, before going live.
- OTA-sourced bookings: upsells are collected directly in OwnerRez as an owner-managed charge and payment, never through the OTA's alteration or resolution flow, and channel-managed charge lines are left untouched.
- API use is rate-limited per IP; the worker uses a pre-emptive limiter plus backoff with jitter.

Related: [[ownerrez-state-is-source-of-truth]], [[native-triggers-for-staff-alerts]].


## Timeline

- time: 2026-09-18T18:27:02
  kind: decision
  summary: "Created this page: Use the OwnerRez v2 API wherever it reaches; drive the web UI only for the three gaps"
  source: "SKILL.md; references/capability-matrix.md; commit 0bbc946"
  affects: [api-first-browser-only-for-gaps]

- time: 2026-09-18T18:27:03
  kind: decision
  summary: "captured from project history: API-vs-UI split and the UI quirks behind it"
  source: "references/capability-matrix.md; commit 0bbc946"
  affects: [api-first-browser-only-for-gaps]

- time: 2026-09-18T18:27:33
  kind: decision
  summary: "lead with a one-line summary so the index entry is informative; content unchanged"
  source: same as previous entry
  affects: [api-first-browser-only-for-gaps]
