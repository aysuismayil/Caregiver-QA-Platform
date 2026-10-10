# BUG-006: Duplicate booking and charge after connection is restored during checkout

**Title:** [MOBILE] Duplicate booking and charge are created after the connection is restored during checkout

**Short description:** The native app automatically retries failed network requests when the connection comes back. If the "confirm booking" request is in progress when the connection drops, the app sends it again as a new request after reconnecting, without checking if the first one already succeeded. This can create two confirmed bookings and two charges from one checkout.

**Preconditions:**
- iOS or Android native app (staging build) on a real device.
- Customer test account logged in, with a saved test payment method.
- Airplane mode can be toggled on the device.
- A provider with one open slot.

**Steps to reproduce:**
1. In the native app, log in as customer.mobile.qa@example.com, choose an open slot and go to the final "Pay Now" step.
2. Tap "Pay Now".
3. Within about 1 second, turn on Airplane mode to drop the connection during the request.
4. Wait 5 seconds, then turn Airplane mode off.
5. Don't tap "Pay Now" again. Watch what the app does on its own after reconnecting.
6. Check "My Bookings" for duplicates.
7. Run the duplicate-payment SQL check ([Query 2](../database-validation/sql-validation-queries.md)) and check the customer's bookings in Postman.

**Actual result:** In 4 of 8 attempts (iOS and Android, about 50%), the app retried the confirm request after reconnecting without checking the first one. "My Bookings" showed two confirmed bookings for the same slot, and SQL and Postman confirmed two payment records and two charges.

**Expected result:** After reconnecting, the app first checks whether the original request completed (for example, with the same idempotency key as in BUG-003) before sending it again. If it already succeeded, the app shows the existing booking instead of creating a new one.

**Test data:**
- Customer: customer.mobile.qa@example.com
- Devices: a real iOS device and a real Android device
- Test card: 4242 4242 4242 4242
- Booking: 2 hours, $50.00

**Environment:** Staging. Network drop simulated with the device's Airplane mode.

**Evidence:** Screenshots of duplicate bookings on iOS and Android, the Postman results and the SQL output were attached to the original ticket. Not included here because they may contain confidential information.

**Severity:** Critical. Same impact as BUG-003 (duplicate charge), with a mobile-only trigger: the app's own automatic retry. It needs its own fix even after BUG-003 is fixed.

**Priority:** P0. Blocks the release. Mobile is a main platform for this product.

**Repro rate:** About 50% (4/8) on both platforms when the connection drops within 1 second of paying. Timing-dependent.

**Workaround:** If it reaches production, customers should check "My Bookings" after any checkout interrupted by a connection drop and contact support for a refund. This only limits the damage.

---

**Investigation notes:**
- Same root cause as BUG-003 (no idempotency on confirm), but a different trigger. In BUG-003 the user retries after seeing an error. Here the app retries in the background without the user doing anything, so the user may never know.
- The staging build's debug logs show a general "retry on reconnect" queue, and the booking confirm call is part of it.
- Suggested fix, together with BUG-003: when the backend supports an idempotency key, the retry queue reuses the same key. Or remove the confirm call from automatic retry and check the booking status after reconnecting instead.
- Recommended retesting together with the BUG-003 fix, because the backend fix should also close this path. Flagged this dependency in triage.
