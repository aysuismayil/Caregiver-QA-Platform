# BUG-005: Customer is charged a different price than shown at checkout

**Title:** [BOOKING] Customer is charged a different price than shown at checkout

**Short description:** The checkout screen loads the provider's hourly rate once and never refreshes it. If the provider changes the rate while the customer is still on checkout, the customer is charged the new rate, an amount they never saw on screen.

**Preconditions:**
- Staging environment.
- Provider test account (PRV-3010) with an editable hourly rate, starting at $25/hr.
- Customer test account with a valid test payment method.
- Two browser sessions: one as the provider, one as the customer.

**Steps to reproduce:**
1. As the provider (PRV-3010), check that the rate is $25/hr. Keep the session open.
2. As the customer, open PRV-3010's profile and start checkout for a 2-hour booking. The summary shows 2 hr × $25/hr = $50.00.
3. Don't pay yet.
4. In the provider session, change the rate from $25/hr to $40/hr and save.
5. Go back to the customer's checkout screen (don't refresh) and click "Pay Now". The screen still shows $50.00.
6. Check the amount actually charged: in the booking confirm response, in Postman (booking details endpoint) and in the database ([Query 3](../database-validation/sql-validation-queries.md)).

**Actual result:**
- The checkout screen never updated. It still showed $50.00 when "Pay Now" was clicked. No request refreshed the rate before submitting.
- The confirm response returned a total charged of 80.00 (2 hr × the new $40/hr rate).
- Postman and the database both confirmed 80.00. The customer was charged $80.00 after only seeing $50.00.

**Expected result:** One of two behaviors, agreed with the PM and designer:
1. The price is locked when the customer enters checkout, like the slot hold (see BUG-002), or
2. If the rate changes during checkout, the customer sees "Price has changed, please review" with the new total before paying.

Charging a different amount than shown, without telling the customer, is not acceptable.

**Test data:**
- Provider: PRV-3010, rate changed from $25.00/hr to $40.00/hr during checkout
- Customer: customer.price.qa@example.com
- Booking: 2 hours
- Total shown at submit: $50.00
- Total charged (API and database): $80.00

**Environment:** Staging. Chrome (DevTools Network and Console). Postman with the staging environment. Read-only PostgreSQL access through DBeaver.

**Evidence:** Screenshots of the checkout screen, DevTools captures of the rate request and the confirm response, the Postman response and the SQL result were attached to the original ticket. Not included here because they may contain confidential information.

**Severity:** High. No data corruption or security exposure, but a customer can be charged an amount they never saw or approved.

**Priority:** P1. Fix before the next release. Not P0 because it needs a provider to change the rate during another customer's live checkout. A narrower case, but real.

**Repro rate:** 5/5 (100%) when the rate change happens between loading checkout and paying.

**Workaround:** None for the customer. As a smaller, faster fix, I suggested blocking rate edits while a provider has an active checkout in progress.

---

**Investigation notes:** I checked three layers to find where the mismatch comes from.
- **Frontend (DevTools):** the checkout screen fetches the rate once on page load and keeps it locally. No refresh before "Pay Now".
- **API (Postman):** the booking details endpoint returns the real charged amount ($80.00), based on the rate at confirmation time. The backend uses the live rate correctly. This is a gap between frontend and backend, not a calculation bug: the backend expects the frontend to re-check the price, and it doesn't.
- **Database:** the payment record matches the API response, so 80.00 was really charged (not a cached value).
- Suggested fix: short term, re-fetch and show the rate right before enabling "Pay Now" and block payment if it changed. Longer term, lock the price together with the slot hold.
