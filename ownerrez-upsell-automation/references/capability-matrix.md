# OwnerRez API vs. UI capability matrix

OwnerRez publishes a documented v2 REST API, but it is not a complete surface — several operations that matter for upsell automation exist only in the web UI. This file lists what is actually true, with enough of the why that you can reason about edge cases it does not explicitly cover.

## The three UI-only operations

**Rezzy tasks have no API.** There is no task resource in the documented v2 API — nothing to list, read, filter, or resolve a task through. If your worker's trigger is "a Rezzy task exists asking about an upsell," that discovery step has to happen in the browser, full stop.

**Adding a charge line to an existing booking is UI-only.** The booking write model (`BookingEditModel` in OwnerRez's terms) does not expose a charges field — only things like arrival, departure, check-in/out time, guest, notes, property, and title. There is no documented endpoint for appending a line item to a booking's ledger. This has to go through Booking → Charges → Change Charges → Add Charge in the UI.

**Submitting a payment is UI-only; reading one is not.** The payments endpoint is GET-only in the documented API — you can list and read payments for a booking, but there is no POST to submit a new one. Actually charging a card on file has to go through Booking → Transactions → Payments → Collect Payment in the UI. The read side, though, is genuinely useful: always verify a payment landed by reading it back through the API rather than trusting the UI's own success indication after submission.

## What is writable via the documented API

- Booking arrival, departure, check-in time, check-out time, notes, title, property assignment
- Calendar blocks — a block is simply a booking created with `is_block: true`; this is how a night-based upsell (an "extra night" that is not literally moving the guest's own checkout date) is typically implemented
- Guest and property reads
- Availability checks
- Payment reads (see above — use this to verify, never to submit)

Prefer the API for all of the above. Falling back to browser automation for something the API already supports just adds fragility for no reason.

## UI quirks that will cost you real time if you do not know about them

**Percentage tax lines do not recalculate when a new charge line is appended.** OwnerRez appends new charge lines at the bottom of the ledger. If a tax line above computes a percentage of the lines above it, a new line appended below is simply never included in that calculation — even though it visually sits "under" the tax line and looks like it should be covered. If a new line needs to be taxed, it has to be repositioned above the tax lines (drag it there in the UI), or you need to independently verify the booking's total balance changed by exactly the intended amount before proceeding to payment, and stop if it did not.

**Never click "Reset to Quote Charges" or "Reset to Property Rates."** Both wipe every existing charge line on the booking, not just the ones related to whatever you were trying to fix. There is no confirmation step that makes this obviously destructive in the moment — it just clears the ledger.

**The "Schedule Payments?" checkbox in Collect Payment defaults to checked.** If left checked, the payment gets scheduled rather than collected immediately, which is not what an upsell charge usually wants. This has to be explicitly unchecked, and — per the general discipline in this skill — re-verified as unchecked by reading the control's actual state, not just clicking it once and trusting the click landed, before submitting.

**Card-on-file detection must look at the Cards On File section specifically, not payment history.** A booking's payment history showing a prior card charge does not mean OwnerRez currently holds a usable stored payment token for that card. This is a real trap on OTA-originated bookings in particular: Airbnb and Vrbo bookings frequently show a card payment in history that was actually processed by the OTA itself, off-platform, with no token ever stored in OwnerRez. Check the Cards On File section under the booking's Transactions tab directly before assuming a charge can be collected.

**Airbnb only supports on-the-hour check-in and check-out times.** If a worker sets a half-hour check-in time (say, 3:30 PM) on an Airbnb-sourced booking, Airbnb's own sync will silently round it to the nearest hour on its side, so what OwnerRez stores and what the guest actually sees/paid for can diverge. If you are accepting a specific time as part of a paid early check-in/late check-out upsell, either constrain the offer to on-the-hour times for Airbnb bookings or explicitly confirm the rounding is acceptable to the guest.

**Do not create OTA alterations, charges, or resolution requests for OTA-sourced bookings.** For Airbnb, Vrbo, and similar channel-managed bookings, upsells should be collected directly in OwnerRez as an owner-managed charge and payment, not routed through the OTA's own alteration/resolution flow. Channel-managed charge lines should be left untouched — mixing owner-managed and channel-managed charges on the same booking is a common source of reconciliation confusion later.

## Rate limits

OwnerRez publishes a limit of 300 requests per 5 minutes per IP address. The API returns a 429 on violation with no documented `Retry-After` header, so do not rely on reading one. Apply exponential backoff with jitter on a 429, and prefer a pre-emptive sliding-window limiter in the worker itself so you rarely hit the limit in the first place rather than reacting to it after the fact.
