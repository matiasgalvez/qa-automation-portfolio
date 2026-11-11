# Mobile Test Strategy

Mobile testing has failure modes web testing does not: interruptions, permissions, flaky networks, OS fragmentation, app store releases you cannot hotfix in minutes. My mobile experience includes an **iOS point-of-sale app** (PIN login, menu management, ticket handling, payments), **in-app purchase and card/PayPal payment flows across multiple countries**, and automation with **XCUITest (iOS)** and **Appium (Android)**.

## Objectives

- Critical journeys work on the devices and OS versions our users actually have.
- The app survives real-world conditions: bad network, interruptions, backgrounding, low battery.
- Payments and data are correct and never duplicated.
- Releases are safe, because a bad build can stay on user devices for weeks.

## Device and OS Coverage

I build the matrix from analytics, not from what is on my desk.

| Tier | Coverage | Example |
|---|---|---|
| **Tier 1** — every build | Top devices ≈ 70% of users, latest OS | iPhone 15/16 (iOS 18), Pixel 8, Samsung Galaxy S24 (Android 14/15) |
| **Tier 2** — release candidate | Older but supported, small and large screens | iPhone SE, iPad, mid-range Samsung A series, Android 12 |
| **Tier 3** — major releases | Oldest supported OS, edge form factors | Minimum OS version, foldables, tablets in landscape |

Simulators and emulators for fast feedback; **real devices (BrowserStack or in-house) for release candidates**, because camera, biometrics, push notifications, performance and payment sheets behave differently on hardware.

## Mobile-Specific Risk Areas

| Area | What I test |
|---|---|
| **Interruptions** | Incoming call, notification, alarm, Control Center, switching apps mid-flow (especially mid-payment) |
| **Lifecycle** | Background → foreground, app killed by the OS, cold vs warm start, state restored correctly |
| **Network** | Offline, airplane mode toggled mid-request, slow 3G, Wi-Fi ↔ cellular switch, request retries without duplicates |
| **Permissions** | First-time prompt, denied, denied permanently, revoked in Settings while app is open |
| **Device state** | Low storage, low battery / power saving, dark mode, large text / Dynamic Type, RTL languages |
| **Orientation & screens** | Rotation, split view, notch / Dynamic Island, small screens |
| **Upgrades** | Upgrade from previous version keeps login and local data; DB migrations |
| **Push & deep links** | Open the right screen from cold start and when already running, logged out vs logged in |

## Payments and POS

Payment flows get the deepest coverage:

- Card, PayPal and in-app purchases per supported country (currency, tax, formatting).
- Declined card, insufficient funds, expired card, 3-D Secure challenge, user cancels the payment sheet.
- **Interruption during payment**: kill the app after tapping Pay → on relaunch the order state is correct and the user is not charged twice.
- Receipt / ticket matches the backend order.
- POS-specific: PIN login and lockout, multiple staff sessions, ticket edits after sending to kitchen, offline mode and sync when connection returns.

## Backend Visibility

Most mobile bugs are really API or data bugs. During testing I watch the traffic and logs:

- **AppSpector** to inspect network requests, responses and local storage on the device
- **Kibana / Datadog** to correlate app actions with backend logs
- Proxy tools to simulate slow or failed responses

## Automation Approach

| Layer | Tool | What |
|---|---|---|
| iOS UI | XCUITest | Critical flows; fast and stable, runs in Xcode / CI on simulators |
| Android UI | Appium (UiAutomator2) | Critical flows; same page-object structure as web |
| Cross-platform | Appium + WebdriverIO | Shared test logic when the apps behave the same |
| Real devices | BrowserStack App Automate | Release-candidate smoke on Tier 1 devices |
| API | Same API suite as web | Business rules covered once, below the UI |

Principles:
- Use accessibility identifiers as selectors (agreed with developers, also improves accessibility).
- Keep mobile UI suites small: login, main purchase path, settings that affect money or data.
- Reset app state between tests; seed data through the API instead of through the UI.

```javascript
// Appium + WebdriverIO page object (Android)
class LoginScreen {
  get pin()      { return $('~pin-input'); }      // accessibility id
  get submit()   { return $('~login-button'); }
  get error()    { return $('~login-error'); }

  async loginWithPin(pin) {
    await this.pin.setValue(pin);
    await this.submit.click();
  }
}
module.exports = new LoginScreen();
```

---

## iOS / Android Regression Checklist

### Install and launch
- [ ] Fresh install launches; onboarding shown once
- [ ] Upgrade from previous store version keeps session and data
- [ ] Cold start under target time on Tier 1 device

### Authentication
- [ ] Login, logout, session expiry
- [ ] Biometric login (Face ID / fingerprint) enabled, disabled, failed
- [ ] PIN lockout after N failed attempts

### Core flows
- [ ] Main purchase / order flow end to end
- [ ] Payments: success, decline, cancel, interruption
- [ ] Data created on mobile appears correctly on web and backend

### Platform behavior
- [ ] Background / foreground during each critical step
- [ ] Offline → online sync, no duplicates
- [ ] Push notification opens the correct screen
- [ ] Deep links (cold and warm)
- [ ] Permissions: allow, deny, revoke

### UI
- [ ] Small screen (iPhone SE / compact Android)
- [ ] Tablet and landscape
- [ ] Dark mode, large text
- [ ] Localization: long translations, currency and date formats

---

## Example Mobile Defects (illustrative)

These are written in the format I use; the scenarios are representative examples, not copied from client projects.

**M-01 — Order charged twice when the app is backgrounded during payment (iOS)**
Steps: add item → tap Pay → immediately press Home → return after 10 s → tap Pay again when the button re-enables.
Expected: one charge, order confirmation shown on return.
Actual: two payment requests in AppSpector, two charges in the payment sandbox.
Severity: Blocker. Root cause area: missing idempotency key on retry.

**M-02 — Ticket edits lost after network reconnect (POS, iPad)**
Steps: turn off Wi-Fi → edit an open ticket (add 2 items) → turn Wi-Fi on.
Expected: edits sync to backend.
Actual: ticket reverts to server version; local edits discarded without warning.
Severity: Critical.

**M-03 — Checkout button hidden behind keyboard on small screens (Android, 360 px wide)**
Expected: screen scrolls so the focused field and Continue are visible.
Actual: Continue is unreachable until the keyboard is dismissed.
Severity: Major.
