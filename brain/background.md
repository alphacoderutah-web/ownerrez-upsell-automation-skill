---
slug: background
title: Project background
role: project background
updated: "2026-09-18T18:26:49"
---

# Project background

## Why

Automating a guest-approved paid add-on (an extra night, early check-in or late check-out) on OwnerRez means moving real money on a real guest's card. Two facts make that hard to do safely:

- **The mistake is asymmetric.** A charge on the wrong booking, or one run before the guest actually agreed, cannot be quietly undone. The whole design follows from that: dry-run by default, authorization traced back to an explicit guest affirmative, and nothing trusted as done until OwnerRez itself confirms it.
- **OwnerRez's documented API stops short of what an upsell needs.** Reading Rezzy tasks, adding a charge line and collecting a card-on-file payment are UI-only, and the UI has quirks that silently produce a wrong charge, or a staff-alert trigger that looks saved but is not.

This repository packages that platform knowledge and a safety pattern as a Claude Code skill, so that a worker Claude builds for an account gets these details right the first time. The skill says its facts were learned by hitting each failure in practice rather than from OwnerRez's own documentation, and asks that they still be re-verified against the live account, because OwnerRez changes its UI over time.

## Goals

Evidenced in the README, `SKILL.md` and `evals/`:

1. Teach the API-vs-UI capability boundary and the UI quirks that cost real time — [[api-first-browser-only-for-gaps]].
2. Give a stack-agnostic safety architecture — [[live-mode-requires-stacked-gates]], [[charge-authorization-chain]], [[ownerrez-state-is-source-of-truth]].
3. Route staff alerts through OwnerRez's native Triggers + Templates, with no separate SMS vendor — [[native-triggers-for-staff-alerts]].
4. Provide a canary rollout path: dry-run, then one verified live charge, then a full portfolio.
5. Make the skill trigger reliably from its name and description alone, and stay quiet on adjacent work — [[trigger-description-evals]].

The only measurable success criterion in the repository today is the trigger eval (every query correct on both the train and held-out splits at the last run). Nothing yet measures whether workers built with the skill avoid the failures it describes (see Open questions).

## Non-goals

- **Shipping a working worker.** This is a methodology and platform-knowledge skill. There is no runnable automation to point at an account; Claude builds the worker with the user, in the language and framework they choose.
- **Knowing any account's fees, staff or properties.** Every amount, contact and property detail must come from the account and its owner. The skill tells Claude to verify them from canonical sources and never assume them; that discipline is the point, not a gap to fill later.
- **Other property-management platforms.** It covers OwnerRez's API and web UI as they actually behave, and does not generalise.
- **Adjacent OwnerRez work**, evidenced by the eval set's should-not-trigger cases: nightly rate or pricing changes, listing content and photo captions, fixing the Rezzy messaging assistant's directives, guest-facing message triggers such as a pre-arrival email, one-off manual UI help, payment-processor dispute handling, and general read-only reporting scripts.

## Target user

A Claude Code user who runs, or builds automation for, an OwnerRez-managed vacation-rental business and wants guest-approved add-ons qualified, charged and collected automatically, with staff alerted. Typically someone doing these steps by hand today who is worried about double charges or charging a guest who never agreed. *(Inferred from the skill description and the should-trigger eval queries.)*

## Open questions

- Is there a commitment to re-verify the UI facts as OwnerRez ships UI changes? The repository does not state one.
- Is a wider distribution channel intended beyond the README's clone-and-copy install?
- How would success be measured for workers built with the skill (for example, zero unintended charges in canary runs)? Not defined in the repository.
