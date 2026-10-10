# BUG-001: Provider at exactly 10 miles is missing from search results

**Title:** [SEARCH] Provider at exactly 10 miles is missing from search results

**Short description:** When a customer filters by distance, a provider exactly on the boundary (exactly 10.0 miles for "within 10 miles") does not appear. The acceptance criteria say the boundary value must be included.

**Preconditions:**
- Staging environment with seeded location test data.
- An active provider seeded at exactly 10.00 miles from a fixed test search location.
- The provider offers a service type included in the search.

**Steps to reproduce:**
1. Log in as a customer test account on staging.
2. Search from the fixed test location (synthetic seed address).
3. Apply the distance filter "Within 10 miles".
4. Don't apply any other filters.
5. Look for provider PRV-1042 (seeded at exactly 10.00 miles) in the results.

**Actual result:** PRV-1042 does not appear in "within 10 miles". It appears only when the filter is widened to the next radius option.

**Expected result:** As agreed in the reviewed AC ([requirements review](../docs/02-requirements-and-acceptance-criteria-review.md), Example 1), the radius filter includes the boundary. A provider at exactly 10.00 mi must appear for "within 10 miles".

**Test data:**
- Search location: synthetic test address with fixed coordinates
- Provider PRV-1042 at exactly 10.00 mi (checked separately with a straight-line distance calculation on the seeded coordinates)
- Filter: radius = 10 miles

**Environment:** Staging. Chrome (also reproduced in Safari).

**Evidence:** Screenshots of the results at both radius options and a DevTools capture of the search request and response were attached to the original ticket. Not included here because they may contain confidential information.

**Severity:** Medium. No crash or data loss, but customers don't see valid providers at common round-number distances.

**Priority:** P1. It breaks a signed-off acceptance criterion and affects a common search.

**Repro rate:** 5/5. Also reproduced with a second provider seeded at exactly 5.00 mi and the "within 5 miles" filter.

**Workaround:** Customers can choose a wider radius, but then they also get providers outside the range they wanted.

---

**Investigation notes:**
- Sent the same search in Postman: the provider is missing from the API response too. So it's not a display issue.
- DevTools Network tab: the frontend sends the radius value 10 correctly.
- Providers at 9.99 mi and 9.5 mi appear correctly, so only the exact boundary is affected.
- Hypothesis (not verified in code): the filter may use "less than" instead of "less than or equal to". Flagged for the developer to check.
- This exact case was added as an AC during requirements review.
