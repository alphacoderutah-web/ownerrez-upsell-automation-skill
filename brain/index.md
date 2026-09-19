# Brain Index

_Auto-generated. Last updated 2026-09-19T00:27:34.729Z._

- [api-first-browser-only-for-gaps](pages/api-first-browser-only-for-gaps.md) — category: decision | tags: [ownerrez-api, browser-automation, capability-map] | Use the documented v2 API for everything it supports; browser automation only for Rezzy tasks, adding a charge line and collecting a card pa
- [charge-authorization-chain](pages/charge-authorization-chain.md) — category: decision | tags: [safety, authorization, fees] | Authorized only with a host-quoted price, the guest's own clear charge-me reply and an owner-approved fee rule; anything less goes to a huma
- [live-mode-requires-stacked-gates](pages/live-mode-requires-stacked-gates.md) — category: decision | tags: [safety, dry-run, rollout] | Dry-run is the default; live mutation needs a config flag, a separate execute flag, a narrow allowlist and a charge ceiling, all at once.
- [native-triggers-for-staff-alerts](pages/native-triggers-for-staff-alerts.md) — category: decision | tags: [staff-alerts, triggers, templates] | Staff SMS and email alerts come from OwnerRez Triggers + Templates fired by the worker's natural booking or block event; three UI gotchas ca
- [ownerrez-state-is-source-of-truth](pages/ownerrez-state-is-source-of-truth.md) — category: concept | tags: [safety, verification, idempotency] | A write is done only when a fresh OwnerRez read shows it; this drives idempotent steps, no blind retry, no auto-rollback and a redacted audi
- [trigger-description-evals](pages/trigger-description-evals.md) — category: decision | tags: [evals, triggering] | Twenty queries (12 train, 8 held out) judged from the name and description alone; every query correct on both splits at the last run, 2026-0
