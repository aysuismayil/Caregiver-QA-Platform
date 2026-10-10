# Release Readiness and Test Summary

This is the format I use to give the team a GO / NO-GO recommendation before a release. The report below is reconstructed for this portfolio. Release name, dates and ticket numbers are not real.

---

## Release: Release A (sample)

**QA owner:** QA Engineer (me)
**Release scope:** search radius filter fix, booking concurrency fix, price lock at checkout, accessibility fixes (form labels, contrast, screen reader announcements)

---

## Summary

Testing for this release is **complete**. I recommend **GO**, with two known issues accepted as risk:

- **BUG-004** (Critical, security): open, accepted by the PM and engineering team, fix planned for the next sprint
- **TC-A11Y-010** (Low, accessibility): no announced warning before the checkout hold expires

All P0 test cases pass. The smoke check is green. Regression found no new Critical or High issues.

## Test execution

All planned functional, API and accessibility checks for the release scope were executed on the release candidate build. Every failure was linked to a bug. All fixed bugs were retested on the release candidate. The only failing checks are TC-AUTH-021 (BUG-004) and TC-A11Y-010. Both are accepted known issues listed below.

## Bugs found in this release cycle

| ID | Title | Severity | Priority | Status |
|---|---|---|---|---|
| [BUG-001](../bug-reports/BUG-001-search-radius-boundary-excluded.md) | Search radius boundary excluded | Medium | P1 | Fixed, verified |
| [BUG-002](../bug-reports/BUG-002-double-booking-race-condition.md) | Double booking (race condition) | Critical | P0 | Fixed, verified |
| [BUG-003](../bug-reports/BUG-003-payment-double-charge-retry.md) | Double charge on payment retry | Critical | P0 | Fixed, verified |
| [BUG-004](../bug-reports/BUG-004-session-token-not-invalidated-on-logout.md) | Session token still valid after logout | Critical | P1 | **Open**, accepted known risk, fix planned for next sprint |
| [BUG-005](../bug-reports/BUG-005-price-mismatch-frontend-backend.md) | Checkout price mismatch | High | P1 | Fixed, verified |
| [BUG-006](../bug-reports/BUG-006-mobile-duplicate-charge-on-reconnect.md) | Mobile duplicate charge on reconnect | Critical | P0 | Fixed, verified |

## Known issues accepted for this release

| Issue | Severity | Why it was accepted |
|---|---|---|
| BUG-004: session token still works after logout | Critical (security) | The full fix needs a larger backend change to how tokens are revoked, which could not be done safely in this release. An interim mitigation (shorter token lifetime) reduced the exposure window. The PM and engineering team accepted the risk for this release, and the fix was planned for the next sprint. Holding the release would also have delayed the verified fixes for BUG-002, BUG-003 and BUG-006. |
| [TC-A11Y-010](../accessibility-testing/accessibility-test-cases.md): no announced warning before the checkout hold expires | Low | Narrow case (screen reader users who need more time during checkout). Does not block completing a booking. Planned for the next accessibility pass. |

## Smoke check

✅ **Pass** on the release candidate build (see [smoke checklist](../smoke-and-regression/smoke-test-suite.md)).

## Regression

✅ **Pass.** Full regression across all feature areas. The only issues found were BUG-001 (since fixed and re-verified) and the accessibility item above. The fixes did not introduce new regressions.

## Risk assessment

| Area | Assessment |
|---|---|
| Payments | **Low risk to ship.** BUG-003 and BUG-006 are fixed and re-verified with targeted regression, including the SQL duplicate-payment check after repeated retry attempts. No duplicates found. |
| Booking | **Low risk to ship.** BUG-002 is fixed and re-verified with near-simultaneous Postman Collection Runner requests and the SQL overlapping-bookings check. No overlapping confirmed bookings found. |
| Security (sessions) | **Accepted, monitored risk.** BUG-004 is still open. I recommended a monitoring alert for token use after logout until the fix ships. |
| Accessibility | **Improved, not complete.** Several fixes shipped in this release. The remaining known gap is Low severity. |

## Recommendation

**GO**, on these conditions:

1. Risk acceptance for BUG-004 is documented and signed off by the PM and engineering team.
2. A monitoring alert for token use after logout is added before or right after the release.
3. A smoke check is run on production right after deployment.
