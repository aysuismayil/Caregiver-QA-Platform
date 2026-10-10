# Regression Scope

Run before each release (full pass) and in a smaller, focused form after a bug fix (the affected area plus flows that share the same code). It uses the detailed cases in [test-cases/](../test-cases/). This page is the planning and tracking view, not a copy of the steps.

## Full regression (before a release)

| Area | Test cases |
|---|---|
| Authentication | All of [01-authentication-test-cases.md](../test-cases/01-authentication-test-cases.md) |
| Search and filters | All of [02-search-filtering-test-cases.md](../test-cases/02-search-filtering-test-cases.md) |
| Provider profiles | All of [03-provider-profile-test-cases.md](../test-cases/03-provider-profile-test-cases.md) |
| Booking | All of [04-booking-test-cases.md](../test-cases/04-booking-test-cases.md) |
| Payments | All of [05-payments-test-cases.md](../test-cases/05-payments-test-cases.md) |
| Mobile | All of [06-mobile-test-cases.md](../test-cases/06-mobile-test-cases.md) |
| API | The [Postman collection](../api-testing/postman/Marketplace-API.postman_collection.json) with Collection Runner |
| Accessibility | New or changed screens only, from [accessibility-test-cases.md](../accessibility-testing/accessibility-test-cases.md) |

When time is short, I run P0 and P1 cases first (see [risk-based priorities](../docs/01-test-plan-and-strategy.md#8-risk-based-priorities)) and P2/P3 if time allows.

## Focused regression after a fix (example)

After a fix I don't only retest the one failed case. I also test everything that shares the same code, because fixes often cause new regressions.

**Example: after the fix for [BUG-002](../bug-reports/BUG-002-double-booking-race-condition.md) (double booking):**

| Retest | Why |
|---|---|
| TC-BOOK-004 (the original failed case) | Confirm the bug is fixed |
| TC-BOOK-002, TC-BOOK-003 | Same hold mechanism: normal hold and release still work |
| TC-BOOK-005 | Slot is still released when payment fails |
| TC-BOOK-009, TC-BOOK-011 | Cancellation and rescheduling also change slot state |
| API-BOOK-02 (Postman) | Rerun the near-simultaneous confirm with Collection Runner; it must not succeed twice |
| SQL Query 1 (overlapping bookings) | Zero overlapping confirmed bookings after repeated near-simultaneous attempts |

## Result log (format)

| Date | Scope | Result | Notes |
|---|---|---|---|
| | Full / focused | Pass / Fail | |
