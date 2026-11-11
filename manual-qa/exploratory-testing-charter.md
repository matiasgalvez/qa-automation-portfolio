# Exploratory Testing Charters

Exploratory testing is where I find the defects nobody wrote a test case for. I run it as **session-based testing**: a time-boxed session with a clear mission (the charter), notes taken while testing, and a short debrief. That keeps it disciplined and reportable, not "clicking around".

## Charter Format

```
Explore   <target area>
With      <resources: users, data, tools, devices>
To discover <information: risks, behaviors, defects>
Time box: 60–90 min
```

## Heuristics I Lean On

- **SFDIPOT** (Structure, Function, Data, Interfaces, Platform, Operations, Time) to make sure I am not only testing the happy path.
- **Boundaries**: empty, one, many, max length, max+1, negative, special characters, emoji, leading/trailing spaces.
- **Interruptions**: refresh, back button, double click, two tabs, session expiry, network drop mid-request.
- **Personas and roles**: new user, returning user, locked user, user with a slow connection, keyboard-only user.
- **Compare with a baseline**: same flow with a known-good user, browser or previous build.
- **Follow the data**: what the UI shows vs what the API sent vs what was stored.

---

## Charter 1 — Checkout across user roles

```
Explore   the cart and checkout flow
With      every SauceDemo test user (standard, problem, error, performance_glitch, visual),
          Chrome DevTools (console + network)
To discover role-specific defects in validation, totals and order completion
Time box: 90 min
```

### Session Notes (real session, saucedemo.com, Nov 2026)

| Time | Note |
|---|---|
| 00:05 | Baseline with `standard_user`: login ~40 ms, checkout completes, totals add up. |
| 00:15 | `problem_user`: every product image is the same 404 placeholder (`sl-404…jpg`). |
| 00:20 | `problem_user`: 3 of 6 Add to cart buttons do nothing — **BUG-04**. |
| 00:27 | `problem_user`: clicking Backpack opens Fleece Jacket (id=5) — **BUG-05**. |
| 00:35 | `problem_user`: Last Name text lands in First Name — **BUG-03**. |
| 00:48 | `error_user`: Last Name does not bind **but Continue still advances** — **BUG-01** (client validation bypassed). Need API check. |
| 00:55 | `error_user`: Finish does nothing, two TypeErrors in console — **BUG-02**. |
| 01:05 | `performance_glitch_user`: login ~5 s, 3 runs — **BUG-06**. |
| 01:15 | `visual_user`: inventory prices don't match baseline — **BUG-07**. |
| 01:25 | Sorting: no-op for `problem_user`, native alert for `error_user` — **BUG-08**. |

### Debrief

- **Bugs:** 8 (2 Blocker/Critical in the revenue path). Full reports in `bug-report-samples.md`.
- **Risks raised:** client-only validation (BUG-01) suggests the API may accept incomplete orders → added an API negative test to the plan.
- **Questions for the team:** is there a server-side price source of truth? Which price reaches the order payload for `visual_user`?
- **Coverage gaps found:** automated suite only used `standard_user` → proposed data-driven checkout tests across all roles.
- **Not covered (next session):** mobile viewports, keyboard-only checkout.

---

## Charter 2 — Checkout form boundaries and input handling

```
Explore   checkout step one (First Name, Last Name, Postal Code)
With      boundary data set, copy/paste, autofill, keyboard only
To discover validation gaps, data truncation and error-message problems
Time box: 60 min
```

Ideas to cover:
- Whitespace-only values (`"   "`) — accepted as valid?
- 1 character, 255 characters, 256 characters
- Names with accents and apostrophes: `José`, `O'Brien`, `Zoë`
- Emoji and non-Latin scripts
- Postal codes: letters, symbols, leading zeros, 3 vs 10 characters
- Paste from clipboard including a trailing newline
- Browser autofill populating every field correctly
- Submitting with Enter vs clicking Continue
- Error message is announced to screen readers and focus moves to the first invalid field

---

## Charter 3 — Responsive layout against Figma

```
Explore   inventory, cart and checkout pages
With      Chrome device emulation (375, 768, 1280 px), BrowserStack iPhone + Pixel, Figma file
To discover layout, overflow and touch-target issues on mobile and tablet
Time box: 60 min
```

Checklist while exploring:
- Text truncation and overflow in product names and prices
- Burger menu opens, closes and traps focus correctly
- Touch targets at least 44×44 px
- Sticky elements do not cover the Continue / Finish buttons
- Landscape orientation
- Zoom to 200% without horizontal scrolling

---

## Turning Sessions Into Value

Every session ends with three outputs: **bugs filed**, **risks or questions** sent to the team, and **automation candidates**, the checks worth repeating on every build. Charter 1 above produced all three.
