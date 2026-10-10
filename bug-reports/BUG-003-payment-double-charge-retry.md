# BUG-003: Customer is charged twice after retrying a failed payment

**Title:** [PAYMENT] Customer is charged twice after retrying a failed payment

**Short description:** When the payment request times out on the customer's screen but actually succeeds on the server, the customer sees an error and tries again. The retry creates a second charge for the same booking instead of recognizing that the first payment already went through.

**Preconditions:**
- Staging environment with network throttling available (Chrome DevTools Network tab).
- Customer test account with a valid test payment method.
- A provider with an open slot.

**Steps to reproduce:**
1. Log in as a customer test account, choose an open slot and go to checkout.
2. Simulate a dropped response: the request reaches the server and succeeds, but the client never gets the answer. (On staging this was done with an artificial server-side delay and cutting the client connection, coordinated with a backend developer for this test.)
3. Submit payment.
4. The client times out and shows "Something went wrong, please try again".
5. Click "Pay Now" again with the same payment details.
6. Check the customer's booking history and the number of payment records for this booking attempt.

**Actual result:** Two payment records are created, and two charges go through on the same test card for what the customer saw as one booking attempt. The confirmed booking appears only after the second charge. The first charge (which actually succeeded) is not cleanly linked to a booking.

**Expected result:** The retry is recognized as the same booking attempt (using an idempotency key created at the start of checkout and reused on retry) and no second charge is created. The original payment and booking are returned to the client, or, if a new attempt is really needed, the first charge is voided first.

**Test data:**
- Customer: customer.retry.qa@example.com
- Test card: 4242 4242 4242 4242, exp 12/29, CVC 123
- Booking: 2 hours, $60.00 total
- Simulated delay: 8-second server-side delay on the confirm call with a 5-second client timeout, to reproduce "client gave up, server kept going"

**Environment:** Staging. Chrome, DevTools network throttling. Payment provider in sandbox/test mode.

**Evidence:** A DevTools network capture of the timed-out request and the retry, the database payment records and the Postman reproduction were attached to the original ticket. Not included here because they may contain confidential information.

**Severity:** Critical. Customers can be charged twice for one service.

**Priority:** P0. Blocks the release. Payments are a P0 risk area in the test plan.

**Repro rate:** About 80% (8/10) with the artificial timeout. Without it, it's harder to trigger on purpose, but support had reports from real users on unstable mobile networks. That is what started this investigation.

**Workaround:** Support can find and refund duplicate charges when a customer reports them, but nothing detects them early. I suggested a temporary monitoring query ([Query 2](../database-validation/sql-validation-queries.md)) to catch duplicates before the customer notices.

---

**Investigation notes:**
- The "Pay Now" button does disable after a click, so this is not a simple double-click bug (that case is covered by TC-PAY-003). This is the timeout-then-retry path.
- In the Network tab, neither the original request nor the retry includes an idempotency key or request ID, so the server can't tell that the retry is the same attempt.
- The payment provider's public API documentation supports an idempotency key header. It was not used in this integration yet.
- Suggested fix: create a key at the start of checkout, send it on the confirm request, and have the backend check it before charging. A retry with the same key then returns the original result.
- Flagged as a permanent regression test, because it depends on exact timing and could break again without anyone noticing.
