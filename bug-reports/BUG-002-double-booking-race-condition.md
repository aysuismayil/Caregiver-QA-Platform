# BUG-002: Two customers can book the same time slot

**Title:** [BOOKING] Two customers can book the same time slot

**Short description:** The slot should be held during checkout and confirmed only after payment. When two customers pay for the same slot at almost the same time, both payments can go through and both bookings can be confirmed. Both customers are charged for a slot only one of them can get.

**Preconditions:**
- Staging environment.
- One provider (PRV-2005) with exactly one open slot: Saturday 10:00 AM–12:00 PM.
- Two customer test accounts, each with a valid test payment method.
- Two separate browser sessions, or one browser session plus Postman.

**Steps to reproduce:**
1. As Customer A, open PRV-2005's profile, choose the Saturday 10–12 slot and go to the final checkout screen. Don't pay yet.
2. As Customer B, in another session, open the same slot's checkout screen. Don't pay yet.
3. Submit payment for both customers within about the same second. (Reproduced manually with two browser sessions, and more precisely with two near-simultaneous Postman requests to the booking confirm endpoint using Collection Runner.)
4. Check the booking status for both customers.
5. Run the SQL check for overlapping confirmed bookings ([Query 1](../database-validation/sql-validation-queries.md)) to confirm at the data level.

**Actual result:** In 3 of 10 timed attempts, both bookings were confirmed for the same slot and both test cards were charged. The SQL check found two `confirmed` rows for the same provider with overlapping times.

**Expected result:** As agreed in the reviewed AC ([requirements review](../docs/02-requirements-and-acceptance-criteria-review.md), Example 2), only the first customer to pay gets a confirmed booking. The second customer sees "This time is no longer available", is not charged, and the slot disappears from search.

**Test data:**
- Provider: PRV-2005, one open slot (synthetic future date on staging)
- Customer A: customer.a.qa@example.com, test card 4242 4242 4242 4242
- Customer B: customer.b.qa@example.com, test card 4000 0566 5566 5556 (a different test card to rule out card-specific caching)
- Requests sent within about 300 ms of each other (checked with request timestamps)

**Environment:** Staging. Reproduced through the UI (two browser sessions) and through the API (Postman Collection Runner), which rules out a frontend button issue.

**Evidence:** Postman run results, SQL query output and screenshots of both customers' bookings were attached to the original ticket. Not included here because they may contain confidential information.

**Severity:** Critical. Two customers pay for a service only one can receive, and the provider is double-booked.

**Priority:** P0. Blocks the release. Booking integrity is a P0 risk area in the test plan.

**Repro rate:** Intermittent, about 30% (3/10) when the requests are sent within about 300 ms. This is expected for a timing issue.

**Workaround:** None for customers or support. As a short-term step, I suggested a monitoring alert for overlapping confirmed bookings for the same provider, so support can catch and fix them until the bug is fixed.

---

**Investigation notes:**
- The timing suggests both requests check "is this slot free?" before either one saves its booking, and then both confirm.
- During a timed test I checked the hold records in the database: a hold is created for the first request, but the second request's confirm call doesn't always check for an existing hold before charging. The hold check and the payment confirmation don't seem to happen as one step.
- I suggested a database-level unique rule for confirmed bookings (same provider and start time) as a hard guarantee, in addition to the application check.
- This kind of bug is easy to miss with manual testing alone. I recommended a dedicated concurrency test.
- This bug led to adding an explicit concurrency AC in requirements review. The original story had none.
