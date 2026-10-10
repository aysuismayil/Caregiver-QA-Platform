# Requirements and Acceptance Criteria Review

I review user stories **before** development starts, in backlog refinement. The goal is to find missing edge cases, unclear wording and untestable acceptance criteria (AC) while they are still cheap to fix.

Below are two examples from my work, generalized for this portfolio. Numbers in the revised AC are example values.

---

## My review checklist

Before I give a story a QA "ready for development":

- [ ] Each AC is a pass/fail statement (not "should work well").
- [ ] Error and negative paths are defined, not only the happy path.
- [ ] Boundary values are clear (min/max, zero results, exact limits).
- [ ] Behavior is defined when two users act at the same time.
- [ ] Validation rules are written (required fields, formats, limits).
- [ ] Web and mobile behavior are both covered.
- [ ] New UI has accessibility expectations.
- [ ] Network failure, API error and timeout behavior is defined.
- [ ] "Done" is clear enough that QA can verify it without guessing.

---

## Example 1: Provider search radius filter

**Original story**

> As a customer, I want to search for providers near me so that I can find someone convenient to book.
>
> **AC:**
> - User can enter a location and see nearby providers.
> - User can filter by distance.

**Gaps I raised**

1. "Nearby" and "distance" are not defined. Miles or km? What is the default radius?
2. No AC for **zero results**.
3. No AC for **invalid location** (unknown address, empty field, location permission denied).
4. No AC for a provider **exactly on the boundary** (exactly 10.0 miles with "within 10 miles"). Included or not?
5. No AC for how the radius works together with other filters (price, rating).
6. Straight-line distance or driving distance? This changes which providers appear.

**Revised AC**

- If the user doesn't choose a radius, a default radius is used (example value: 20 miles).
- Distance is straight-line distance in miles and is shown on each provider card (for example, "3.2 mi away").
- The radius filter **includes** the boundary value: a provider at exactly 10.0 mi appears for "within 10 miles".
- If the location can't be found, show an inline error: "We couldn't find that location. Try a city, ZIP code, or address."
- If no providers match, show an empty state with a hint to widen the radius or clear filters. Never a blank page or endless spinner.
- The radius filter combines with other filters using AND logic.

Later in testing, I found that the boundary rule was not implemented correctly: [BUG-001](../bug-reports/BUG-001-search-radius-boundary-excluded.md).

---

## Example 2: Booking a time slot

**Original story**

> As a customer, I want to book a provider for a specific date and time so that I can schedule a service.
>
> **AC:**
> - User selects a date/time and confirms the booking.
> - Provider receives a notification.

**Gaps I raised**

1. No AC for **double booking**: what if two customers book the same slot at almost the same time?
2. What does "confirm" lock in? Is the slot held during checkout, or only after payment?
3. No AC for **cancellation** windows or fees.
4. No AC for provider **decline or no response**. Is the booking automatic, or must the provider accept it?
5. No AC for **time zones** (customer and provider can be in different time zones).
6. No AC for what happens if payment fails **after** the slot looks booked.

**Revised AC**

- When a customer starts checkout, the slot is **held for a short time** (example value: 5 minutes) so nobody else can book it.
- The booking becomes `confirmed` only after payment succeeds. If payment fails, the slot is released right away and the customer sees a clear error with a retry option.
- If two customers reach checkout for the same slot, the second one to pay sees "This time is no longer available" and goes back to search.
- Providers accept or decline within a response window (example value: 12 hours). If they don't respond, the booking is cancelled automatically and the customer is notified and refunded.
- Booking times are stored in UTC and shown in each user's local time zone with a label (for example, "2:00 PM EST").

Later in testing, I found that the slot hold did not prevent double booking when two requests came in at almost the same time: [BUG-002](../bug-reports/BUG-002-double-booking-race-condition.md).

---

## Why this matters

Both gaps later became bugs found in testing: BUG-001 (Medium) and BUG-002 (Critical). Because the rules were written down during review, the bugs were easy to prove and the expected behavior was not in question.
