# Caregiver QA Platform (Web + Mobile)

QA portfolio based on my professional experience testing a two-sided **service-booking marketplace** (web, iOS and Android). In this repository the product is called **the Marketplace App**.

> **About this repository**
> The QA work, methods and types of issues are real. To protect the client's confidential information, the product name, data, endpoints, database structure, release names and business rules are removed, generalized or reconstructed. Business values in the requirements and test cases (prices, time windows, distances, fees) are example values. Evidence files (screenshots, screen recordings, network captures, Postman results, SQL output) are not included because they may contain confidential information.

---

## The product (generalized)

The Marketplace App connects **service providers** with **customers**:

- **Providers** create a profile (bio, services, hourly rates, availability calendar) and get found in search.
- **Customers** search for providers by location, service type, price, rating and availability, book a time slot and pay in the app.

Flows I tested: sign-up and login, provider profiles, search and filters, availability and booking, payments and refunds, notifications, and mobile (native apps + mobile web).

---

## My role

I was the Manual QA Engineer on a cross-functional team (2 backend developers, 2 frontend developers, 1 PM, 1 designer). I:

- reviewed user stories and acceptance criteria in refinement, before development started
- wrote test plans, entry/exit criteria and test cases for web and mobile
- ran manual functional, smoke and regression testing
- ran exploratory sessions on booking and payment flows
- tested APIs directly with Postman, including near-simultaneous requests with Collection Runner
- checked backend data with read-only SQL queries on staging
- did accessibility checks (WCAG 2.1 AA, keyboard, VoiceOver, TalkBack, axe DevTools)
- reported bugs in Jira, investigated them with DevTools, Postman and SQL, and retested fixes
- prepared release readiness reports and gave GO / NO-GO recommendations

---

## Where to start

| I want to see... | Go to |
|---|---|
| How I plan testing | [docs/01-test-plan-and-strategy.md](docs/01-test-plan-and-strategy.md) |
| How I review requirements before development | [docs/02-requirements-and-acceptance-criteria-review.md](docs/02-requirements-and-acceptance-criteria-review.md) |
| My release readiness report | [docs/03-release-readiness-and-test-summary.md](docs/03-release-readiness-and-test-summary.md) |
| How I approach testing this kind of product | [docs/04-testing-approach.md](docs/04-testing-approach.md) |
| Test cases | [test-cases/](test-cases/) |
| Bug reports | [bug-reports/](bug-reports/) |
| Exploratory testing | [exploratory-testing/](exploratory-testing/) |
| API testing (Postman) | [api-testing/](api-testing/) |
| SQL data validation | [database-validation/](database-validation/) |
| Accessibility testing | [accessibility-testing/](accessibility-testing/) |
| Smoke and regression | [smoke-and-regression/](smoke-and-regression/) |
| Templates | [templates/](templates/) |

---

## Repository contents

| Folder | What's inside |
|---|---|
| `docs/` | Test plan, requirements review, release readiness, testing approach |
| `test-cases/` | 103 functional test cases in 6 areas (authentication, search, provider profiles, booking, payments, mobile) |
| `accessibility-testing/` | 12 accessibility checks with results |
| `bug-reports/` | 6 bug reports (BUG-001 to BUG-006) |
| `exploratory-testing/` | Session charter with notes, and a session template |
| `api-testing/` | API test cases and a sample Postman collection |
| `database-validation/` | Read-only SQL queries used to confirm backend data |
| `smoke-and-regression/` | Smoke checklist and regression scope |
| `templates/` | Bug report and test case templates |

The test cases are written for this portfolio to show my test design. The count is the content of this repository, not the number of test cases executed on the real project.

---

## Bugs in this repository

| ID | Title | Severity | Priority | Status |
|---|---|---|---|---|
| [BUG-001](bug-reports/BUG-001-search-radius-boundary-excluded.md) | Provider at exactly 10 miles is missing from search results | Medium | P1 | Fixed, verified |
| [BUG-002](bug-reports/BUG-002-double-booking-race-condition.md) | Two customers can book the same time slot | Critical | P0 | Fixed, verified |
| [BUG-003](bug-reports/BUG-003-payment-double-charge-retry.md) | Customer is charged twice after retrying a failed payment | Critical | P0 | Fixed, verified |
| [BUG-004](bug-reports/BUG-004-session-token-not-invalidated-on-logout.md) | User can access the account after logging out | Critical | P1 | Open, accepted known risk |
| [BUG-005](bug-reports/BUG-005-price-mismatch-frontend-backend.md) | Customer is charged a different price than shown at checkout | High | P1 | Fixed, verified |
| [BUG-006](bug-reports/BUG-006-mobile-duplicate-charge-on-reconnect.md) | Duplicate booking and charge after connection is restored | Critical | P0 | Fixed, verified |

---

## Tools

- **Bug tracking and documentation:** Jira, Confluence, Google Docs/Sheets
- **API testing:** Postman (requests, test scripts, Collection Runner)
- **Data validation:** SQL (PostgreSQL), DBeaver, read-only staging access
- **Browser investigation:** Chrome DevTools (Network, Console, Application)
- **Mobile:** real iOS and Android devices, BrowserStack
- **Accessibility:** axe DevTools, VoiceOver, TalkBack, keyboard-only testing
- **Version control:** Git

---

**Aysu Ismayilzada** · QA Engineer
[GitHub](https://github.com/aysuismayil) · [Portfolio](https://aysuismayil.github.io)
