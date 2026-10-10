# Test Cases: Mobile (iOS, Android, Mobile Web)

Native app behavior compared to web, responsive layouts, device features and gestures. Columns are explained in the [test case template](../templates/test-case-template.md).

## Coverage

| Platform | Coverage |
|---|---|
| iOS native app | Real iOS devices; BrowserStack for other device/OS combinations |
| Android native app | Real Android devices; BrowserStack for other device/OS combinations |
| Mobile web | Safari on iOS, Chrome on Android; small screen widths (example: 375 px) |

## Test cases

| ID | Title | Preconditions | Test Data | Steps | Expected Result | Priority | Type | Platform |
|---|---|---|---|---|---|---|---|---|
| TC-MOB-001 | Full booking flow on the iOS app | iOS app installed, logged in | Test card | Search → book → pay | Booking confirmed; same behavior and data as web | P0 | Positive | iOS |
| TC-MOB-002 | Full booking flow on the Android app | Android app installed, logged in | Test card | Search → book → pay | Booking confirmed; same behavior and data as web | P0 | Positive | Android |
| TC-MOB-003 | Push notification for booking confirmation | Push permission granted | — | Complete a booking with the app in the background | Push arrives with correct details; tapping it opens the right booking | P1 | Positive | iOS/Android |
| TC-MOB-004 | Location permission request is clear | Fresh install | — | Open search for the first time | System prompt with a clear reason; denying it doesn't crash the app and manual location entry still works | P1 | Positive/Negative | iOS/Android |
| TC-MOB-005 | Search works after denying location | Location permission denied | — | Deny permission, then search | User can type a location and search normally | P1 | Negative | iOS/Android |
| TC-MOB-006 | Network drop during booking | In checkout | Network drop (Airplane mode) | Start checkout, drop the network during the request, reconnect | No duplicate charge after reconnecting (see [TC-PAY-004](05-payments-test-cases.md), [BUG-006](../bug-reports/BUG-006-mobile-duplicate-charge-on-reconnect.md)); user sees a retry/offline state, not a crash or freeze | P0 | Negative/Edge | iOS/Android |
| TC-MOB-007 | App in background during checkout | In checkout | Hold time (example: 5 minutes) | Put the app in the background, come back after 2 minutes (within the hold time) | User returns to the same step with the slot still held; after the hold time, a clear "session expired, please restart" message | P1 | Positive | iOS/Android |
| TC-MOB-008 | Layout works on small screens | Mobile web, 375 px | — | Open search results, profile and checkout at 375 px | No horizontal scrolling, no cut-off text or buttons, tap targets usable | P1 | Positive | Mobile web |
| TC-MOB-009 | Large system font size | Device font set to the largest size | — | Increase the system font size, open the app | Text scales and layout adjusts without cut-off or overlap | P1 | Positive/Accessibility | iOS/Android |
| TC-MOB-010 | Camera and photo library for profile photo | Provider logged in on the native app | — | Tap "Upload photo", try "Take Photo" and "Choose from Library" | Both work; permission prompts appear when needed; photo uploads | P2 | Positive | iOS/Android |
| TC-MOB-011 | Deep link opens the right screen | App installed | Link to a provider profile | Tap a shared link (SMS/email) | App opens on that provider's profile, not just the home screen | P2 | Positive | iOS/Android |
| TC-MOB-012 | Screen rotation | On any screen | — | Rotate the device on search results and in checkout | Layout adjusts without losing form input or crashing | P2 | Positive | iOS/Android |
| TC-MOB-013 | Biometric login with password fallback | Biometrics enabled | — | Log in with Face ID/fingerprint, then test the fallback | Biometric login works; if it fails, password login is offered | P2 | Positive | iOS/Android |
| TC-MOB-014 | Old app version shows "please update" | Old app build against the current backend | Old build | Open an outdated build on staging | If a minimum version is enforced, user sees a clear "please update" screen | P2 | Negative | iOS/Android |
| TC-MOB-015 | Native app and mobile web show the same data | Same account on the app and mobile browser | Same booking | Open the same booking on both | Price, time and status are identical | P1 | Positive | iOS/Android/Mobile web |
