# Trigger-description evaluation

Checks that Claude Code, seeing only this skill's `name` and `description` —
never its body — reliably consults it for in-scope requests and reliably
leaves it alone for out-of-scope ones. `trigger-eval.json` holds the 20
queries: 12 for tuning (`split: "train"`), 8 held out and used only to score
the final description (`split: "test"`), so a revision can't just overfit to
the exact phrasings it was tuned against.

Coverage is deliberately adversarial. The should-trigger queries vary in tone
and specificity (formal, casual, typo'd, never naming the skill outright).
The should-not-trigger queries are near-misses on purpose — several
deliberately collide with adjacent OwnerRez work this skill does *not* cover:
Rezzy guest-messaging directive fixes, PriceLabs/OwnerRez nightly-rate
changes, listing content and photo captions, a one-off manual UI question, a
Stripe chargeback question, a general read-only reporting script, and (the
trickiest one) a guest-facing pre-arrival-email Trigger — same underlying
OwnerRez mechanism this skill documents, wrong purpose.

## Results (2026-08-18)

| | Train (12) | Test (8) |
|---|---|---|
| Original description | 12/12 (100%) | 8/8 (100%) |

The description scored perfectly on the first pass, so no revision was
needed or made — 0 improvement rounds.

## Methodology note

The skill-creator plugin's own description-optimization tool
(`scripts/run_loop.py`) shells out to a standalone `claude` CLI binary via
subprocess for each query. That binary was not present on the machine this
skill was built on — it runs inside the Claude desktop app's local-agent-mode
sandbox rather than a terminal `claude-code` install — so the official script
could not run there.

The methodology was instead replicated directly: for each query, a fresh
subagent was given *only* the skill's name and description (exactly what
Claude Code sees in `available_skills` before deciding whether to consult a
skill) plus the query, with no knowledge of this skill's actual body or of
any other context, and asked whether it would consult the skill. This is
the same underlying question the official tool measures — it just uses an
isolated subagent call instead of a subprocess to a separate CLI process.

To rerun or extend this: for each entry in `trigger-eval.json`, ask a fresh,
context-isolated Claude instance (or run the skill-creator plugin's own
`scripts/run_loop.py` on a machine with the `claude` CLI installed) to
decide, from the name and description alone, whether it would consult this
skill for that query. Compare against `should_trigger`.
