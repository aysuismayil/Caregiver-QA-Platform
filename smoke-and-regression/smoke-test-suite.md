# Smoke Checklist

Run right after every deploy to staging, and on production right after a release, before wider regression. Goal: a quick check that the critical paths work. If any check fails, I stop and escalate right away. There is no point running regression on a build where login is broken.

**When:** every deploy to staging; right after a production release
**Owner:** QA (me)

| # | Check | Steps | Expected | Platform |
|---|---|---|---|---|
| 1 | App/site loads | Open the site / app | Loads without errors, no blank screen or 500 page | Web/Mobile |
| 2 | Customer sign up | Create a customer account with valid data | Account created, correct next screen | Web |
| 3 | Customer login | Log in with a known staging test account | Logged in, lands on home | Web/Mobile |
| 4 | Provider login | Log in with a known provider test account | Logged in, lands on provider dashboard | Web/Mobile |
| 5 | Search returns results | Search a known location with seeded providers | Results list is not empty | Web/Mobile |
| 6 | Filters work | Apply one filter (for example service type) | Results narrow down, no error | Web/Mobile |
| 7 | Provider profile loads | Open a provider from the results | Photo, bio, rate and availability load | Web/Mobile |
| 8 | Checkout opens | Choose an open slot and go to payment | Checkout loads with the correct price | Web/Mobile |
| 9 | Test card payment | Pay with the standard test card | Booking confirmed, confirmation shown | Web/Mobile |
| 10 | Booking appears in "My Bookings" | Open booking history | New booking with correct details | Web/Mobile |
| 11 | Cancellation works | Cancel the booking just created | Status becomes cancelled, refund started | Web/Mobile |
| 12 | Logout works | Log out | Session ends, login/landing page shown | Web/Mobile |
| 13 | Native apps start | Cold-start the iOS and Android apps | No crash, login screen shown | iOS/Android |
| 14 | Third-party services respond | Trigger a location lookup and a test-mode payment | Both respond correctly | Web/Mobile |

## Result log (format)

| Date | Build | Result | Notes |
|---|---|---|---|
| | | Pass / Fail at # | |
