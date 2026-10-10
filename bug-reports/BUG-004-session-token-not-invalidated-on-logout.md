# BUG-004: User can access the account after logging out

**Title:** [ACCOUNT] User can access the account after logging out

**Short description:** Logging out should end the session on the server, not only on the screen. A token captured before logout can still be used to call protected API endpoints after the user logs out. The server does not revoke the token; only the UI looks logged out.

**Preconditions:**
- Staging environment.
- Customer test account, logged in on the web app.
- The auth token can be captured with Chrome DevTools (Network tab on an authenticated request).
- Postman available to replay the request outside the browser.

**Steps to reproduce:**
1. Log in as customer.session.qa@example.com on staging web.
2. In DevTools → Network, open the customer's booking history page and copy the bearer token from that request.
3. In Postman, call the same protected booking-history endpoint with that token. It returns 200 OK with booking data. (This confirms the token works before logout.)
4. In the browser, click "Log Out".
5. Confirm the UI returns to the login page.
6. Send the same Postman request from step 3 again, with the same token.

**Actual result:** The request in step 6 still returns 200 OK with the customer's booking data. The token stayed valid after logout for as long as I tested: up to 15 minutes. I did not test beyond 15 minutes.

**Expected result:** Logout ends the session on the server immediately. A request with a token from before logout returns 401 Unauthorized.

**Test data:**
- Account: customer.session.qa@example.com
- Token: a bearer token captured before logout (the value is never included in documentation)
- Endpoints: a protected booking-history endpoint, also reproduced on a protected profile endpoint

**Environment:** Staging. Chrome. Postman used to replay the request, which rules out a browser cache effect.

**Evidence:** Postman results before and after logout (token value hidden) and a DevTools capture of the logout call were attached to the original ticket. Not included here because they may contain confidential information.

**Severity:** Critical. Session security issue: a captured token (from a shared computer, a compromised device or a browser extension) keeps working after the user thinks they are logged out.

**Priority:** P1. Urgent, fix in the next sprint. Not release-blocking for this release because the PM and engineering team formally accepted the risk (see [release readiness](../docs/03-release-readiness-and-test-summary.md)).

**Repro rate:** 5/5 (100%). The token was still valid when retested up to 15 minutes after logout, the longest window I tested.

**Workaround:** None for end users. As an interim step, I suggested shortening the access token lifetime to make the exposure window smaller while the real fix is built.

---

**Investigation notes:**
- In DevTools → Application, logout does clear the token on the client. So the logout button works; the problem is that the server doesn't revoke the token, so any separate copy of it (like my Postman request) still works.
- The logout response (200 OK) doesn't show whether the server revoked anything. I noted this as a separate, smaller issue.
- Hypothesis (not verified in code): tokens are not checked against a server-side revocation list. Two possible fixes: a server-side check on each request, or short-lived access tokens with a refresh token that can be revoked.
- Added as a permanent regression test ([TC-AUTH-021](../test-cases/01-authentication-test-cases.md)) and suggested a security review.
