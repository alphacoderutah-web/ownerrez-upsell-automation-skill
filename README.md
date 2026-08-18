# OwnerRez Upsell Automation Skill

A Claude Code skill for building safe, auditable guest-upsell automation on
**OwnerRez** — extra nights, early check-in, late check-out, or similar paid
add-ons — qualified from a guest's own words, charged and collected through
OwnerRez, with staff notified automatically.

A charge run on the wrong booking, or run before the guest actually agreed to
it, cannot be quietly undone. That asymmetry shapes the whole design: dry-run
is the default, authorization has to trace back to an unambiguous guest
affirmative, and nothing gets trusted as done until it is re-read back from
OwnerRez itself.

This is a **methodology and platform-knowledge skill**, not a bundled
application. It teaches Claude the specific, hard-won facts about how
OwnerRez actually behaves — where its documented API stops and browser-only
UI takes over, and the exact UI quirks that silently produce a wrong charge —
so that whatever worker Claude builds for your account gets these right the
first time instead of the hard way.

## What you get

- **The API-vs-UI capability map** — which OwnerRez operations (Rezzy tasks,
  adding a charge, collecting a payment) have no documented API and require
  browser automation, and which UI quirks bite: tax lines that don't
  recalculate for a newly appended charge, a "Schedule Payments?" checkbox
  that defaults the wrong way, Airbnb's on-the-hour check-in/out rounding,
  and how to actually confirm a card is on file.
- **A safety architecture** — dry-run by default, a multi-factor live-mode
  gate, an authorization chain that requires an explicit guest affirmative
  (never inferred from a bare "yes"), an idempotent state machine that never
  blind-retries an ambiguous charge, and an audit trail.
- **Native staff alerts, no SMS vendor required** — how to wire up OwnerRez's
  own Triggers + Templates system to text or email staff on booking events,
  including three specific bugs in the Trigger editor's UI that will silently
  eat your configuration if you don't know about them.
- **A pre-launch checklist** — the canary-rollout sequence for taking a
  worker from dry-run to its first verified live charge, and from one booking
  to a full portfolio.

## Install

```bash
git clone https://github.com/alphacoderutah-web/ownerrez-upsell-automation-skill
cp -r ownerrez-upsell-automation-skill/ownerrez-upsell-automation ~/.claude/skills/
```

Or drop `ownerrez-upsell-automation/` into a project's `.claude/skills/`
instead of your personal one.

## Triggering

Claude decides whether to consult a skill from its name and description
alone, before reading the body — so those need to hold up on their own.
`evals/` has a 20-query test set (12 tuning, 8 held out) checking that this
description reliably triggers on in-scope requests and reliably stays quiet
on adjacent ones — including deliberate near-misses against rate-setting,
listing content, photo captions, and Rezzy guest-messaging work. Current
description: 100% on both splits. See `evals/README.md` for the full
methodology and how to rerun or extend it.

## What it doesn't do

- **Ship a working worker.** There is no pre-built automation here to point
  at your account — Claude builds it with you, in your language and
  framework of choice, using the facts and patterns this skill provides.
- **Know your fees, staff, or properties.** Every dollar amount, phone
  number, and property detail has to come from your own account and your own
  business owner. The skill explicitly instructs Claude to verify these from
  canonical sources rather than assume or invent them — that discipline is
  the point, not a gap to fill in later.
- **Cover non-OwnerRez platforms.** This is specific to OwnerRez's API and
  web UI as they actually behave; it doesn't generalize to other property
  management systems.

## Licence

MIT. No warranty — it is a guide for building something that moves real
money on a real guest's card. Start in dry-run, verify against OwnerRez
itself before trusting any result, and canary one booking before widening.
