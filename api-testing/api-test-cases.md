# API Test Cases (Postman)

I test the API directly with Postman, without the UI. This shows whether a bug is in the frontend or the backend, covers negative and edge cases quickly, and checks contract behavior.

The sample collection is in [postman/Marketplace-API.postman_collection.json](postman/Marketplace-API.postman_collection.json) with an environment file in [postman/Marketplace-Environment.postman_environment.json](postman/Marketplace-Environment.postman_environment.json).

> Endpoint paths, IDs, URLs and field names in this folder are illustrative. They do not match the real API.

## Why I test at the API level

- **Speed:** many negative checks against one endpoint take minutes, not hours of clicking.
- **Isolation:** shows whether a bug is frontend or backend. In [BUG-005](../bug-reports/BUG-005-price-mismatch-frontend-backend.md), Postman showed the backend charged the live price and the frontend never refreshed it.
- **Cases the UI can't reach easily:** invalid payloads, missing headers, wrong IDs, boundary values and near-simultaneous requests. [BUG-002](../bug-reports/BUG-002-double-booking-race-condition.md) was reproduced with Postman Collection Runner.
- **After a fix:** rerunning the collection quickly confirms the fix and checks related endpoints before a full UI pass.

## Collection structure

| Folder | Covers |
|---|---|
| `Auth` | Login, logout, wrong password, reusing a token after logout |
| `Search` | Radius search, boundary case, invalid radius |
| `Booking` | Slot hold, booking confirm, nonexistent slot, near-simultaneous confirm |
| `Payments` | Payment confirm, retry without idempotency key, changed amount from the client |

## Example test cases

| ID | Request | Scenario | Expected | Priority |
|---|---|---|---|---|
| API-AUTH-01 | `POST /auth/login` | Valid credentials | `200`, returns an access token and a refresh token | P0 |
| API-AUTH-02 | `POST /auth/login` | Wrong password | `401`, generic error, no hint about which field is wrong | P0 |
| API-AUTH-03 | `GET` protected booking-history endpoint | Token reused after logout (see [BUG-004](../bug-reports/BUG-004-session-token-not-invalidated-on-logout.md)) | Should be `401`; currently `200` (bug) | P1 |
| API-SRCH-01 | `GET /providers/search?radiusMiles=10` | Provider at exactly 10.00 mi (see [BUG-001](../bug-reports/BUG-001-search-radius-boundary-excluded.md)) | Provider included; was excluded (bug, fixed) | P1 |
| API-SRCH-02 | `GET /providers/search?radiusMiles=-5` | Negative radius | `400 Bad Request` with a validation message, not a server error | P2 |
| API-BOOK-01 | `POST /checkout/hold` | Open slot | `200`, hold created with an expiry time | P0 |
| API-BOOK-02 | `POST /checkout/confirm` | Two near-simultaneous confirms on the same slot (see [BUG-002](../bug-reports/BUG-002-double-booking-race-condition.md)) | Only one succeeds; the other gets a clear conflict | P0 |
| API-BOOK-03 | `POST /checkout/confirm` | Slot ID that doesn't exist | `404 Not Found`, no stack trace in the response | P1 |
| API-PAY-01 | `POST /payments` | Retry without an idempotency key (see [BUG-003](../bug-reports/BUG-003-payment-double-charge-retry.md)) | Retry is treated as the same attempt, no second charge | P0 |
| API-PAY-02 | `POST /payments` | Client sends a lower `amount` than the server price | Server calculates the price itself and ignores or rejects the client value | P0 |
| API-PAY-03 | `POST /payments/refund` | Refund for a booking cancelled late | `200`, correct partial refund per policy, itemized | P1 |

## How the Postman tests are written

Each request has a **Tests** script that checks the status code, the response shape and key values, not just "something came back". Example:

```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response has expected booking fields", function () {
    const body = pm.response.json();
    pm.expect(body).to.have.property("bookingId");
    pm.expect(body).to.have.property("status");
    pm.expect(body.status).to.be.oneOf(["pending_provider_response", "confirmed"]);
});

pm.test("Total charged matches expected calculation", function () {
    const body = pm.response.json();
    pm.expect(body.totalCharged).to.eql(pm.environment.get("expectedTotal"));
});
```

The login request saves the tokens to environment variables in its Tests script, so the other requests can use them. The collection can be run with Collection Runner for a quick regression pass after a deploy.
