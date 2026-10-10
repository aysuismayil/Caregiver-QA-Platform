# Testing Approach: How I Test the Marketplace App

This page explains how I think about testing this product and why I spend more time on some areas than others. The [test plan](01-test-plan-and-strategy.md) covers scope, environments and criteria. This page covers the thinking behind it.

---

## What the product must get right

The Marketplace App is a two-sided marketplace: providers list services and availability, customers search, book and pay. Two things matter most:

- **Customers must trust it with their money.** No double charges, no surprise prices.
- **Bookings must be reliable.** No double bookings, no lost bookings, no stale search results.

So payments, booking and authentication get the most testing time. A bug there costs money and trust at the same time.

## Where I start: the requirements

Before a story reaches development, I review it in refinement with a checklist: testable AC, negative paths, boundary values, two users at the same time, web and mobile behavior. The [requirements review](02-requirements-and-acceptance-criteria-review.md) shows two real examples. In both, a gap I raised later became a real bug in testing (BUG-001 and BUG-002). The revised AC gave me a clear expected result to test against.

## How I cover each layer

| Layer | What I do | Where |
|---|---|---|
| UI, web and mobile | Structured test cases for every feature area, positive, negative and boundary | [test-cases/](../test-cases/) |
| Critical path | Smoke check after each deploy: search, profile, checkout, sign up, login, logout | [smoke-and-regression/](../smoke-and-regression/) |
| End to end | The full booking flow: search with radius → filters → profile → slot → checkout → confirmation | [booking](../test-cases/04-booking-test-cases.md), [mobile](../test-cases/06-mobile-test-cases.md) |
| API | Postman requests, independent of the UI, to check contracts and edge cases the UI hides | [api-testing/](../api-testing/) |
| Data | Read-only SQL on staging to confirm that what the UI shows is what was really saved | [database-validation/](../database-validation/) |
| Mobile | Real iOS and Android devices plus BrowserStack, including network drops and backgrounding | [mobile test cases](../test-cases/06-mobile-test-cases.md) |
| Accessibility | Keyboard, VoiceOver, TalkBack, axe DevTools, WCAG 2.1 AA contrast | [accessibility-testing/](../accessibility-testing/) |
| Unknowns | Exploratory sessions with a charter, two users at once, interruptions | [exploratory-testing/](../exploratory-testing/) |

## Where the bugs came from

None of the six bugs came from the happy path. Each one came from a specific edge case:

| Edge case | Bug |
|---|---|
| A provider exactly on the radius boundary | [BUG-001](../bug-reports/BUG-001-search-radius-boundary-excluded.md) |
| Two customers paying for the same slot at nearly the same time | [BUG-002](../bug-reports/BUG-002-double-booking-race-condition.md) |
| Retrying a payment after an unclear failure | [BUG-003](../bug-reports/BUG-003-payment-double-charge-retry.md) |
| Reusing a token after logout | [BUG-004](../bug-reports/BUG-004-session-token-not-invalidated-on-logout.md) |
| Price changed by the provider during checkout, checked in UI, API and database | [BUG-005](../bug-reports/BUG-005-price-mismatch-frontend-backend.md) |
| Network drop during payment on mobile | [BUG-006](../bug-reports/BUG-006-mobile-duplicate-charge-on-reconnect.md) |

The pattern: in a marketplace that handles money, the most valuable edge cases are **two users and the same resource at the same time**, **frontend and backend disagreeing**, and **failure and retry**. Invalid form input matters too, but it rarely causes the most serious bugs.

## Not covered here

Load and performance testing were not part of this work.
