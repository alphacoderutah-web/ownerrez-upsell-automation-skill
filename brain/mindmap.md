---
slug: mindmap
title: Feature mindmap
role: feature mindmap
updated: "2026-09-18T18:26:49"
---

# Feature mindmap

## Feature mindmap

The skill's knowledge branches, as organised across `SKILL.md`, its three reference files and `evals/`.

```mermaid
mindmap
  root((OwnerRez upsell automation skill))
    Platform knowledge
      API writable
        Check-in and check-out times
        Calendar blocks as bookings
        Booking fields and reads
        Payment reads for proof
      UI only
        Rezzy tasks
        Add charge line
        Collect card payment
      UI quirks
        Tax lines skip appended charges
        Reset buttons wipe the ledger
        Schedule Payments defaults checked
        Cards On File, not payment history
        Airbnb on-the-hour times
        OTA bookings collected in OwnerRez
      Rate-limit backoff with jitter
    Safety architecture
      Dry-run by default
      Stacked live-mode gates
      Explicit guest authorization
      Owner-approved fees only
      Idempotent resumable steps
      No blind retry
      No automatic rollback
      Redacted audit trail
    Staff alerts
      Trigger pairs Event with Action
      Typed Templates and merge fields
      Upsell mapped to natural event
      Optional Tag condition
      Three save gotchas
      Canonical contacts only
    Rollout
      One-booking canary
      Verify in OwnerRez
      Widen booking, property, account
    Trigger evals
      Train and held-out splits
      Adversarial near-misses
```

Pages behind the branches: [[api-first-browser-only-for-gaps]], [[live-mode-requires-stacked-gates]], [[charge-authorization-chain]], [[ownerrez-state-is-source-of-truth]], [[native-triggers-for-staff-alerts]], [[trigger-description-evals]].
