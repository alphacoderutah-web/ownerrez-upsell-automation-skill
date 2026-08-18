# Pre-launch safety checklist

A terse, canary-style checklist for taking an upsell worker from dry-run to its first live charge, and from one booking to a full rollout. Work through it in order — each step exists because skipping it is how the earlier incidents that produced this skill actually happened.

1. **Every fee amount going live is confirmed by the business owner, in writing, for every property/service pair.** Never an invented or estimated number, and never one borrowed from a similar-looking property. No approved rule for a given pair means that case routes to human review — it does not mean picking the closest number you can find.

2. **Confirm the Add Charge and Collect Payment form field labels against the live account before going live.** These are two of the three UI-only operations and the most consequential to get wrong, since both move real money. Confirm the labels and layout without actually clicking Save or Submit while doing this check.

3. **Every staff phone number or contact is confirmed against a canonical source** — a working Template's To field, a Team/Staff record, or the person directly — never inferred from message history. Explicitly check for and flag any guest/staff name collision before trusting a number tied to a name.

4. **Pick exactly one booking for the first live run.** Set an allowlist scoped to that single booking, and a charge ceiling set just above the one expected amount — not a round number set far above it, since the ceiling's job is to bound the blast radius of a bug, not to accommodate every hypothetical future charge.

5. **Confirm a real card on file, via the Cards On File section specifically**, not payment history, before running the canary. See `capability-matrix.md` for why payment history is not sufficient evidence here.

6. **Run the single canary live, then verify success in OwnerRez itself, not in the worker's own logs.** Confirm exactly one new charge line, one payment, correct booking state, and — if a Trigger is involved — that the staff alert actually fired, either by checking the Trigger's own send history or by asking the recipient directly.

7. **Reconcile via the documented payments-read endpoint before any second attempt on an ambiguous outcome.** Never blind-retry a payment whose result you are not certain of — confirm nothing already landed before submitting again.

8. **Only after one clean, fully-verified canary, widen the allowlist deliberately.** Move from a single booking to a property-level allowlist, then to the full account, as separate steps rather than jumping straight from one booking to everything.
