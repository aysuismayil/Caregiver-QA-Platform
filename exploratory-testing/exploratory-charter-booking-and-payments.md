# Exploratory Testing Charter: Booking and Payment Flow

I use session-based exploratory testing (charter + time box + notes) next to scripted test cases, especially where scripted cases can't predict every real sequence of user actions. This is the format I use. The session below is reconstructed for this portfolio.

## Charter

**Explore** the end-to-end booking and payment flow (search → profile → book → pay → confirmation), including interruptions, going back, and fast or unexpected actions
**With** a customer test account, a provider test account in a second session that I control, and Chrome DevTools open the whole time
**To discover** problems with consistent state, two users acting at once, and error handling that scripted test cases might miss

**Time box:** 90 minutes · **Tester:** QA (me) · **Environment:** Staging

## Session notes

| # | What I tried | What happened | Bug filed? |
|---|---|---|---|
| 1 | Start checkout, press the browser back button several times, then forward | State stayed consistent; the slot hold was still respected | No |
| 2 | Start checkout for the same slot in two tabs with the same customer account | The second tab showed "you already have this slot in progress" instead of creating a second hold. Good handling | No |
| 3 | Start checkout, change the provider's rate in the second session, then pay in the original screen | The price shown did not match the amount charged | Yes: [BUG-005](../bug-reports/BUG-005-price-mismatch-frontend-backend.md) |
| 4 | Add a provider to favorites, deactivate that provider's account, open favorites again | The deactivated provider still showed with a "Book Now" button that led to a blank checkout page instead of a "no longer available" message | Logged as a lower-severity item (not one of the 6 featured bugs) |
| 5 | Toggle several search filters quickly while results are still loading | Sometimes old results flashed before the correct ones loaded. The final data was correct | Logged as a low-priority UI item |
| 6 | Start checkout, then leave it and start checkout with a different provider | The first hold was released; a database check showed no leftover holds | No |
| 7 | On mobile, start checkout, put the app in the background, lock the phone, come back after the hold time has passed | Correct "session expired, please choose a new time" message (matches [TC-MOB-007](../test-cases/06-mobile-test-cases.md)) | No |
| 8 | Double-click "Pay Now" as fast as possible, and tap quickly on mobile | Led to the duplicate-charge investigation; two different root causes found | Yes: [BUG-003](../bug-reports/BUG-003-payment-double-charge-retry.md), [BUG-006](../bug-reports/BUG-006-mobile-duplicate-charge-on-reconnect.md) |
| 9 | Booking note with emoji, right-to-left text and a very long text (2,000+ characters) | Handled correctly; long text was cut with a visible counter; no crash or broken data | No |
| 10 | Cancel a booking, then right away try to book the same slot again | For about 1–2 seconds the slot still showed as unavailable, then fixed itself without a refresh | Logged as a low-priority note, not a bug |

## Session summary

- Two serious findings that scripted test cases had missed: the price mismatch (BUG-005) and the duplicate-charge pattern that led to BUG-003 and BUG-006.
- Several smaller UX items were logged for the backlog.
- Several "should be fine" assumptions were confirmed: holds across tabs, and mobile backgrounding. Exploratory testing builds confidence too, not only finds bugs.
- Most useful technique in this session: adding **a second user** (the provider acting at the same time as the customer). Some of the most serious bugs on this project (BUG-002, BUG-005) appear only when two users act on the same thing at almost the same time.

## How I choose what to explore

I spend exploratory time on:

- Areas with money or business risk (payments, booking confirmation) before cosmetic areas.
- Two users on the same resource at the same time.
- Interruptions and recovery (network drops, backgrounding, back button, session expiry).
- Areas that just changed. New code is where new bugs are.
