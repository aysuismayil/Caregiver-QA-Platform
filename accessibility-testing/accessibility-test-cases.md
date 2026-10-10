# Accessibility Testing

I checked the main customer flows (search, profile, booking, checkout) against WCAG 2.1 AA. Every customer, including people who use assistive technology, must be able to complete these flows on their own.

## Tools

| Tool | Use |
|---|---|
| axe DevTools (Chrome extension) | First automated scan for common problems (contrast, missing labels, ARIA misuse) |
| Keyboard only (no mouse) | Tab order, visible focus, everything works with the keyboard |
| VoiceOver (iOS/macOS) | Screen reader checks in the native app and Safari |
| TalkBack (Android) | Screen reader checks in the native app and Chrome |
| Browser zoom 200% | Content reflows and stays usable |
| Chrome DevTools contrast checker | Manual contrast checks on custom components |

Automated scans are only a first pass. The manual keyboard and screen reader checks find the problems that really block users.

## Test cases

"Result" is the result of the first pass. The failed items were reported to the team, and the form label, contrast and announcement fixes shipped in the release described in the [release readiness report](../docs/03-release-readiness-and-test-summary.md). TC-A11Y-010 stayed open as a known Low issue.

| ID | Title | Steps | Expected Result | Priority | Result |
|---|---|---|---|---|---|
| TC-A11Y-001 | Full booking flow with the keyboard only | Tab through search → profile → booking → checkout with Tab, Shift+Tab, Enter, Space | Every control can be reached and used; a booking can be completed | P0 | Pass |
| TC-A11Y-002 | Visible focus on all controls | Tab through the app and watch the focus outline | Every focused element has a clearly visible outline | P0 | Pass. The date picker had no visible focus at first; reported and fixed |
| TC-A11Y-003 | Tab order follows the visual layout | Tab through search results and the checkout form | Logical reading order, no jumping around | P1 | Pass |
| TC-A11Y-004 | Form fields have proper labels | Check the checkout fields (name, card form, address) with a screen reader | Each field reads a meaningful label (for example "Cardholder name, edit text"), not only a placeholder | P0 | Fail. Two fields used placeholder text only; reported as an accessibility bug |
| TC-A11Y-005 | Images have meaningful alt text | Check provider photos and icons with a screen reader | Photos read something meaningful (for example "Photo of [provider name]"); decorative icons are hidden from screen readers | P1 | Pass, but one icon-only button (favorite) had no accessible name; reported |
| TC-A11Y-006 | Color contrast meets WCAG AA (4.5:1 normal text, 3:1 large text/UI) | Check prices, buttons and status badges (for example the green "Confirmed" badge) | All checked combinations meet the required ratio | P1 | Fail. The light green "Confirmed" badge text on white measured about 3.2:1, below 4.5:1; reported |
| TC-A11Y-007 | Screen reader announces booking status changes | With VoiceOver/TalkBack on, complete a booking | Status changes ("Booking confirmed") are announced through a live region | P1 | Fail. The confirmation appeared on screen but was not announced; reported |
| TC-A11Y-008 | Error messages are announced and linked to their field | Trigger a validation error (invalid email) with a screen reader on | The error is announced and linked to the field, so it's read again when the field gets focus | P1 | Pass |
| TC-A11Y-009 | Content reflows at 200% zoom | Zoom to 200% on search results and checkout | Nothing cut off, no horizontal scrolling for main content, everything still works | P2 | Pass |
| TC-A11Y-010 | Checkout hold time doesn't disadvantage screen reader users | Start checkout with a screen reader and take longer than average | A warning before the hold expires is announced, not only shown as a silent timer | P2 | Fail. No announced warning before the hold expires; reported |
| TC-A11Y-011 | Dialogs keep focus and close with the keyboard | Open the "Cancel booking" dialog with the keyboard | Focus moves into the dialog, Tab stays inside it, Escape closes it and focus returns to the button | P1 | Pass |
| TC-A11Y-012 | Native app supports large system text | Set iOS/Android font size to the largest setting | Text scales and layout adjusts without cut-off (same goal as [TC-MOB-009](../test-cases/06-mobile-test-cases.md)) | P1 | Pass |

## Summary

This pass found real problems for users: form fields with placeholder-only labels, low contrast on the "Confirmed" badge, and status changes not announced to screen readers. I reported accessibility issues with the same bug report structure and the same care as functional bugs, because they block users from finishing the same flows.
