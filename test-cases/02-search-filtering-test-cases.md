# Test Cases: Provider Search and Filters

Location search, filters, sorting and pagination. Columns are explained in the [test case template](../templates/test-case-template.md). Related bug: [BUG-001](../bug-reports/BUG-001-search-radius-boundary-excluded.md). Default radius, page size and service names are example values.

| ID | Title | Preconditions | Test Data | Steps | Expected Result | Priority | Type | Platform |
|---|---|---|---|---|---|---|---|---|
| TC-SRCH-001 | Search returns providers within the default radius | Seed data with providers at known distances from a test location | Synthetic test location | Enter the location, search with no filters | Providers within the default radius (example: 20 miles) are returned, sorted by relevance/distance | P0 | Positive | Web/Mobile |
| TC-SRCH-002 | Radius filter excludes providers outside the range | Seeded providers at 12.0 mi and 20.0 mi | Filter: within 10 miles | Apply "within 10 miles" | Both providers are excluded (both are outside 10 miles) | P1 | Negative | Web/Mobile |
| TC-SRCH-003 | Radius filter includes a provider exactly on the boundary | Seeded provider at exactly 10.0 mi | Filter: within 10 miles | Apply "within 10 miles" | The provider at exactly 10.0 mi IS included, as the AC says | P1 | Boundary | Web/Mobile |
| TC-SRCH-004 | Filter by service type | Providers offering different services | Filter: "Home Cleaning" | Apply the service filter | Only providers offering Home Cleaning are shown | P0 | Positive | Web/Mobile |
| TC-SRCH-005 | Filter by price range | Providers with different hourly rates | Filter: $20–$35/hr | Apply the price filter | Only providers in the range are shown; providers at exactly $20 and $35 are included | P1 | Boundary | Web/Mobile |
| TC-SRCH-006 | Filter by minimum rating | Providers rated 3.0–5.0 | Filter: 4+ stars | Apply the rating filter | Only providers rated 4.0 or higher are shown | P1 | Positive | Web/Mobile |
| TC-SRCH-007 | Several filters together (AND) | Mixed seed data | Service: Tutoring, Price: $15–$25, Rating: 4+ | Apply all three filters | Results match all three conditions, not just one | P0 | Positive | Web/Mobile |
| TC-SRCH-008 | No results shows a helpful empty state | Filters nobody can match | Rating 5.0 AND price under $5 | Apply the filters | Empty state with a hint to widen the radius or clear filters; not a blank screen or endless spinner | P1 | Edge | Web/Mobile |
| TC-SRCH-009 | Invalid location | None | Location: "asdkjhaskjdh123" | Enter random text as the location, search | Inline error "We couldn't find that location..."; search doesn't run | P1 | Negative | Web/Mobile |
| TC-SRCH-010 | Empty location | No device location permission | Location field blank | Search with no location | User is asked to enter a location or enable location services; no crash or generic error | P2 | Negative | Web/Mobile |
| TC-SRCH-011 | Sort by distance (nearest first) | Providers at different distances | — | Choose "Sort: Nearest" | Results in ascending distance | P1 | Positive | Web/Mobile |
| TC-SRCH-012 | Sort by rating (highest first) | Providers with different ratings | — | Choose "Sort: Top rated" | Results in descending rating; ties sorted consistently (for example by number of reviews) | P2 | Positive | Web/Mobile |
| TC-SRCH-013 | Sort by price (both directions) | Providers with different rates | — | Choose each price sort | Correct order in both directions | P2 | Positive | Web/Mobile |
| TC-SRCH-014 | Pagination loads the next page | More results than one page (example page size: 20) | — | Scroll down / tap "Load more" or page 2 | More providers load with no duplicates or gaps from page 1 | P1 | Positive | Web/Mobile |
| TC-SRCH-015 | Filters stay applied on the next page | Filters applied, several pages | Filter: Home Cleaning | Apply the filter, go to page 2 | Page 2 still respects the filter | P1 | Positive | Web/Mobile |
| TC-SRCH-016 | "Clear all filters" resets the results | Filters applied | Any filters | Tap "Clear all filters" | All filters removed; full results for the location shown | P2 | Positive | Web/Mobile |
| TC-SRCH-017 | Paused providers don't appear in search | One paused provider in range | Paused provider | Search a location that includes the paused provider | The paused provider is NOT shown | P1 | Negative | Web/Mobile |
| TC-SRCH-018 | Availability filter shows only providers free on the date | Mixed availability | Date: a specific Saturday | Filter "Available on [date]" | Only providers with open slots that day are shown | P0 | Positive | Web/Mobile |
| TC-SRCH-019 | Typing in search doesn't send a request for every key | None | Type a location slowly | Watch the Network tab in DevTools while typing | Search/autocomplete requests are debounced, not sent on every key press | P2 | Non-functional | Web |
| TC-SRCH-020 | Special characters don't break the search | None | Location: `TestCity; DROP TABLE users;--` | Enter an SQL-injection-style string, search | Input is handled safely; no server error or unexpected data; treated as an unknown location | P1 | Security/Negative | Web/Mobile |
