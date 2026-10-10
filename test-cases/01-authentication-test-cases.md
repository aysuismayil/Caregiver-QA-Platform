# Test Cases: Authentication

Sign up, login, logout, password reset and session handling for provider and customer accounts. Columns are explained in the [test case template](../templates/test-case-template.md). Policy numbers (password rules, lockout time, link expiry) are example values.

## Sign up

| ID | Title | Preconditions | Test Data | Steps | Expected Result | Priority | Type | Platform |
|---|---|---|---|---|---|---|---|---|
| TC-AUTH-001 | Sign up with valid data (customer) | No account exists for the email | email: customer.qa1@example.com, password: `Str0ngP@ss!`, role: Customer | 1. Open Sign Up 2. Enter name, email, password 3. Select "Customer" 4. Submit | Account created, verification email sent, user sees the "verify your email" screen | P0 | Positive | Web |
| TC-AUTH-002 | Sign up with valid data (provider) | No account exists for the email | email: provider.qa1@example.com, password: `Str0ngP@ss!`, role: Provider | Same as above, select "Provider" | Account created, user goes to provider onboarding (profile setup), not the customer home | P0 | Positive | Web |
| TC-AUTH-003 | Sign up rejected for an existing email | An account exists for the email | Existing email: customer.qa1@example.com | 1. Open Sign Up 2. Enter the existing email 3. Submit | Inline error: "An account with this email already exists. Log in instead?" with a link to Login | P1 | Negative | Web/Mobile |
| TC-AUTH-004 | Password strength validation | None | password: `1234` | Enter a weak password and submit | Inline error with the rules (min 8 characters, 1 number, 1 special character); submit blocked | P1 | Negative | Web/Mobile |
| TC-AUTH-005 | Email format validation | None | email: `not-an-email` | Enter an invalid email and submit | Inline error "Enter a valid email address"; submit blocked | P2 | Negative | Web/Mobile |
| TC-AUTH-006 | Required field validation | None | Name left blank | Submit with the name field empty | Inline error under the name field; submit blocked; no partial account created | P2 | Negative | Web/Mobile |
| TC-AUTH-007 | Terms of Service checkbox is required | None | Valid data, ToS unchecked | Fill all fields, leave ToS unchecked, submit | Submit blocked with a message to accept the ToS | P2 | Negative | Web |

## Login

| ID | Title | Preconditions | Test Data | Steps | Expected Result | Priority | Type | Platform |
|---|---|---|---|---|---|---|---|---|
| TC-AUTH-010 | Login with valid credentials | Verified account exists | customer.qa1@example.com / correct password | Enter valid credentials, submit | User lands on the home screen for their role; session token is set | P0 | Positive | Web/Mobile |
| TC-AUTH-011 | Login rejected with wrong password | Verified account exists | Correct email, wrong password | Enter valid email and wrong password, submit | Generic error "Incorrect email or password" (doesn't say which field is wrong); no lockout after the first attempt | P0 | Negative | Web/Mobile |
| TC-AUTH-012 | Account lockout after repeated failed attempts | Verified account exists | Correct email, wrong password ×5 | Try to log in with the wrong password 5 times in a row | Account is temporarily locked (example policy: 15 minutes) with a message; the lockout message shows even with the correct password until the time passes | P1 | Negative/Security | Web/Mobile |
| TC-AUTH-013 | Login blocked for unverified email | Account created, email not verified | Unverified account | Try to log in | Message "Please verify your email to continue" with a resend option; user is not logged in | P1 | Negative | Web/Mobile |
| TC-AUTH-014 | "Remember me" keeps the session after restart | Valid account | — | Log in with "Remember me", fully close and reopen the app/browser | User is still logged in | P2 | Positive | Web/Mobile |
| TC-AUTH-015 | Provider and customer go to the correct home screen | Both account types exist | Provider and customer accounts | Log in as each role | Provider → provider dashboard (bookings, availability); customer → search/home | P0 | Positive | Web/Mobile |

## Logout and sessions

| ID | Title | Preconditions | Test Data | Steps | Expected Result | Priority | Type | Platform |
|---|---|---|---|---|---|---|---|---|
| TC-AUTH-020 | Logout | Logged in | — | Tap/click Logout | Session ends, user goes to the login/landing page, protected pages need a new login | P0 | Positive | Web/Mobile |
| TC-AUTH-021 | Session token is invalidated on the server after logout | Logged in, token captured with DevTools | Captured bearer token | 1. Log out 2. Replay an authenticated API request with the old token in Postman | API rejects the request with 401 Unauthorized. Currently fails: see [BUG-004](../bug-reports/BUG-004-session-token-not-invalidated-on-logout.md) | P1 | Security/Negative | Web (API) |
| TC-AUTH-022 | Expired session goes to login, not a broken screen | Logged in, session expired (staging setting) | — | Let the session expire, then use the app | User sees login with a clear "session expired" message; no silent failures | P1 | Negative | Web/Mobile |
| TC-AUTH-023 | Logout on one device doesn't log out other devices | Logged in on two devices | — | Log out on device A only | Device B stays logged in and works | P2 | Positive | Web/Mobile |

## Password reset

| ID | Title | Preconditions | Test Data | Steps | Expected Result | Priority | Type | Platform |
|---|---|---|---|---|---|---|---|---|
| TC-AUTH-030 | Request a password reset | Account exists | customer.qa1@example.com | Enter the email on "Forgot password", submit | The same message is shown whether or not the account exists ("If that email exists, a reset link was sent"), so emails can't be guessed; a reset email is sent if the account exists | P0 | Positive/Security | Web/Mobile |
| TC-AUTH-031 | Reset link expires | Reset link older than the allowed time (example policy: 1 hour) | Expired reset link | Open the expired link | Message "This link has expired, request a new one"; password can't be changed | P1 | Negative/Security | Web |
| TC-AUTH-032 | Reset link works only once | Valid, unused reset link | Reset link | 1. Reset the password with the link 2. Open the same link again | Second attempt fails with "This link is no longer valid" | P1 | Negative/Security | Web |
| TC-AUTH-033 | New password must meet the rules on reset | Valid reset link | Weak password: `abc123` | Try to set a weak password through reset | Inline error; reset blocked until the rules are met | P2 | Negative | Web |

## Role-based access

| ID | Title | Preconditions | Test Data | Steps | Expected Result | Priority | Type | Platform |
|---|---|---|---|---|---|---|---|---|
| TC-AUTH-040 | Customer can't open provider-only screens by URL | Logged in as customer | URL of a provider-only page (availability editor) | Open the provider-only URL directly | Access denied or redirected; no provider data or controls shown | P0 | Security/Negative | Web |
| TC-AUTH-041 | Provider can't see another provider's private data by changing the ID | Two provider accounts | Provider A logged in, Provider B's ID | Change a dashboard URL/API call to use Provider B's ID | Request rejected (403); no private data from Provider B | P0 | Security/Negative | Web (API) |
