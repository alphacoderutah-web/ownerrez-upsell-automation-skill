# The Triggers + Templates playbook

OwnerRez ships its own native event-to-action automation system, under Settings > Templates and Settings > Triggers. For staff-facing alerts specifically — "text the property manager when an upsell is confirmed," "text housekeeping when a check-in time changes" — this means you do not need to build or maintain a separate SMS vendor integration inside your worker at all. OwnerRez sends the message itself, from its own configured SMS number, as part of the account's existing subscription.

This is genuinely the most valuable and least-documented thing this skill knows. It also has three gotchas that will each independently produce a trigger that looks correctly saved while silently missing the one field that actually matters. Read this whole file before building anything here — do not try to reverse-engineer the UI live, because the failure modes described below look exactly like success until you specifically know to check for them.

## The core model: a Trigger pairs an Event with an Action

**Events** come in two flavors:

- **Immediate** — fires the instant something happens. For bookings: created, canceled, dates changed, check-in/out time changed, notes changed, payment added. Blocked-Off Time (calendar blocks) has its own parallel set of immediate events: created, updated, deleted.
- **Scheduled** — fires N days or hours before or after an anchor point, such as arrival, departure, or booking-created.

The **Action** is a saved Template — Email, SMS, or Channel Message.

**Conditions** optionally scope which bookings or blocks a trigger applies to. Property is a genuine multi-select, grouped by owner in the picker UI, which is how you route an alert to the correct subset of properties. Dozens of other structured fields exist too — dates, statuses, tags, guest counts. There is, notably, no free-text or content-matching condition. You cannot condition a trigger on "the message body contains X," or on any field that is not a structured, enumerable attribute of the booking or block.

## Template types are strict, and each has its own merge-field vocabulary

A template is typed — Booking, Blocked-Off Time, Guest, Inquiry, Payment, Property, and others — and a Trigger's Action dropdown only offers templates whose type matches that trigger's event category. You cannot wire a Booking-type template to a Blocked-Off-Time event, or vice versa.

Each type's merge fields reflect what data actually exists for that kind of record. A Booking-type template gets fields like the current and the *previous* check-in/check-out time (which is what lets an alert say "changed from 3:00 PM to 5:00 PM" rather than just the new value), arrival/departure dates, booking id, and total amount. A Blocked-Off-Time-type template gets an entirely different, smaller field set — start date, end date, title, reason — because a block is not a guest booking and has no guest or payment data to merge in.

## Mapping an upsell type to the right event, without needing content-matching

Since there is no content-matching condition, do not try to fire a trigger on a generic event like "booking payment added" and then attempt to filter by amount or description — there is no field for that, and you would end up alerting on every payment regardless of relevance.

Instead, pick the event that the mutation itself naturally produces:

- A **time-based upsell** (early check-in, late check-out) is implemented as a change to the booking's own check-in/check-out time, so it naturally corresponds to the "booking check-in/out time is changed" event.
- A **night-based upsell** implemented as a calendar block (rather than moving the guest's own booking dates) naturally corresponds to "Blocked-Off Time is created."

This sidesteps the missing content-matching capability entirely — you are not filtering on *why* the event happened, because the event itself is specific enough to only fire for the case you care about.

If you genuinely need to distinguish an automation-driven mutation from an unrelated manual staff edit that produces the same event, the supported mechanism is a **Tag condition**: have the worker apply a marker tag to the booking as the final step of its own mutation, then scope the trigger's condition to that tag. This works, but it adds an extra write step with a real ordering dependency — the tag has to be set before the event that the trigger's condition evaluates against, which usually means tagging first and mutating second, or accepting a brief window where the tag is not yet present. For most teams, firing unconditionally on the natural event is simpler and arguably more useful anyway, since staff generally want to know about any check-in/out time change or new block regardless of what caused it.

## The three gotchas

Each of these produces the exact same symptom — a trigger that saves, redirects to what looks like a success page, and then turns out to have an empty or wrong Action field when you actually check. They have different root causes, and knowing all three is what turns "why does this keep silently failing" into a five-minute fix instead of an hours-long debugging session.

### Gotcha 1 — the Action dropdown's real state lives in hidden radio buttons, not the visible select

The Action/Template dropdown is a custom-styled widget wrapping a real `<select name="TemplateId">` element, but the widget's own internal state is tracked by a parallel, hidden set of `<input type="radio">` elements — one per template option — rather than by the select itself.

If a value is set programmatically on the select alone (for example, driving the DOM directly via browser automation, bypassing the widget's own click-and-filter UI), the change reads back correctly immediately afterward. It then gets silently overwritten back to empty at save time, because the widget serializes from the radio buttons' checked state at submission, not from the select's current value.

**Symptom:** everything looks correctly set right up until submission, then the server rejects with "The Template field is required" — or worse, silently saves with an empty action and no error at all.

**Fix:** the reliable default is to interact through the actual visible widget — click it open, type into its filter box, click the matching option, the way a person would. If driving the DOM directly is unavoidable, find the specific `input[type=radio][value="..."]` matching the desired option, set `.checked = true` on it, and dispatch both a `change` and a `click` event on that radio element specifically — not the select. Then re-verify the real select's value reflects the change before trusting it.

### Gotcha 2 — changing the Event type asynchronously reloads the Action dropdown's valid options

Switching a trigger's event (for example, from the default "Booking is created" to a different event) kicks off an asynchronous fetch that repopulates the Template/Action select's option list to match the new event's compatible template type. If the Action value is set before this reload lands — even moments before — the selection is silently wiped once the reload completes, even though a same-instant readback showed it as correctly set at the time.

This produces the identical "Template field is required" failure as gotcha 1, through a completely different mechanism, which is what makes it genuinely confusing to debug without knowing both exist independently.

**Fix:** always set the event type first, let it settle — doing other field-setting work in between as a natural buffer works fine, or add an explicit short wait if working purely sequentially — and set the Action/Template last, immediately before submitting. Never the reverse order.

### Gotcha 3 — never trust the save response as proof; reload and re-read

Both gotchas above can each independently produce a save that looks successful — the page redirects to what appears to be a success URL — while the one field that actually matters silently failed to persist. The only reliable verification is to navigate to the record's own edit URL as a fresh page load afterward, not the post-save redirect page, and re-read the actual field values directly. Do not stop at "it redirected somewhere that looked like success."

This generalizes beyond Triggers and Templates specifically — treat it as the default discipline for any mutation made through OwnerRez's UI, not just this one.

## A practical sanity check

Trigger and Template list pages show a running total, something like "Showing X of Y." Noting Y before and after creating a batch of new records is a fast, cheap way to confirm the count landed exactly where intended — no duplicates left behind from a retried failed save, no orphaned partial records.
