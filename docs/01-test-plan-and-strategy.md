# Test Plan and Strategy: the Marketplace App

**Owner:** QA Engineer (manual)
**Scope:** Web (responsive) and mobile apps (iOS, Android) of a service-booking marketplace
**Note:** Portfolio version. Product details are generalized.

---

## 1. Goal

Make sure every release lets customers find a provider, book a slot and pay without double bookings, wrong charges or broken accounts, and catch regressions before customers do.

## 2. In scope

| Area | What is covered |
|---|---|
| Authentication | Sign up, login, logout, password reset, session handling, role-based access (provider vs. customer) |
| Search and filters | Location/radius search, filters (service type, price, rating, availability), sorting, pagination |
| Provider profiles | Create and edit profile, photo upload, certifications, services and rates, availability calendar |
| Booking | Availability, booking creation, rescheduling, cancellation, booking status lifecycle |
| Payments | Checkout through the payment provider's form, receipts, refunds, failed payments |
| Mobile | Native iOS/Android apps and mobile web, same behavior as web |
| Notifications | Booking confirmations, reminders, payment receipts (email and push) |
| Accessibility | Keyboard navigation, screen readers, color contrast on key flows |

## 3. Out of scope

- Load and performance testing
- Internal admin tools
- The payment provider's own systems (we test our integration with it)

## 4. Test types and approach

| Test type | How | Who |
|---|---|---|
| Requirements review | Review user stories and acceptance criteria in refinement, raise gaps and edge cases | QA with PM and developers |
| Functional | Structured test cases on staging before each release | QA (me) |
| Exploratory | Time-boxed sessions with a charter, focused on booking and payments | QA (me) |
| API | Postman requests, independent of the UI | QA (me) |
| Data validation | Read-only SQL queries to confirm backend data matches the UI | QA (me) |
| Accessibility | Keyboard, screen reader and contrast checks on customer flows | QA (me) |
| Smoke | Critical-path check after each deploy to staging and production | QA (me) |
| Regression | Full pass before a release, focused pass after bug fixes | QA (me) |

## 5. Environments

| Environment | Use | Data |
|---|---|---|
| Staging | Main QA environment: manual, API, exploratory and SQL checks | Test data only, payment provider in test mode |
| Production (after release) | Smoke check only, no destructive testing | Real data, read-only observation |

All payment testing uses the payment provider's sandbox/test mode. No real cards or real charges.

## 6. Entry criteria

- The user story has clear, testable acceptance criteria (see [Requirements review](02-requirements-and-acceptance-criteria-review.md)).
- The feature is deployed to staging and passes a basic smoke check.
- Test accounts and test data are ready on staging.
- Third-party sandboxes (payments, maps) are set up and reachable.

## 7. Exit criteria

- All P0/P1 test cases for the release scope have passed, or any failure is linked to a bug with a decision.
- No open Critical or High bugs, unless the risk is formally accepted by the PM and engineering team.
- Smoke check passes on the release candidate build.
- Regression passes on affected areas. Known issues have a workaround or risk note.
- Accessibility spot-check passes on new or changed customer screens.
- A release readiness report is shared (see [release readiness](03-release-readiness-and-test-summary.md)).

## 8. Risk-based priorities

| Priority | Areas | Why |
|---|---|---|
| **P0** | Payments, booking creation and cancellation, authentication | Direct impact on money and customer trust |
| **P1** | Search and filters, availability, profile data | Broken search means fewer bookings |
| **P2** | Notifications, non-critical profile fields, sorting | Important, but has a workaround or smaller impact |
| **P3** | Cosmetic issues that don't block a flow | Fixed when there is time |

The same scale is used for bug priority (see the [bug report template](../templates/bug-report-template.md)).

## 9. Roles

| Role | Responsibility |
|---|---|
| QA Engineer (me) | Test planning, test cases, execution, exploratory testing, API and SQL checks, accessibility, bug reports, release recommendation |
| Developers (2 backend, 2 frontend) | Fix bugs, help with investigation, unit tests |
| Product Manager | Prioritizes fixes, accepts or rejects known risks before release |
| Designer | Confirms expected UI/UX when behavior is unclear |

## 10. Deliverables

- Test cases: [test-cases/](../test-cases/)
- Bug reports: [bug-reports/](../bug-reports/)
- Exploratory charters and notes: [exploratory-testing/](../exploratory-testing/)
- API tests: [api-testing/](../api-testing/)
- Smoke and regression checklists: [smoke-and-regression/](../smoke-and-regression/)
- Release readiness report: [docs/03-release-readiness-and-test-summary.md](03-release-readiness-and-test-summary.md)

## 11. Assumptions

- Staging is configured like production but uses test data.
- Payments, maps and push notifications are tested with their sandbox environments.
- Mobile testing uses real iOS and Android devices plus BrowserStack (see [mobile test cases](../test-cases/06-mobile-test-cases.md)).
