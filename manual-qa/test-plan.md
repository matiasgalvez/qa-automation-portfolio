# Test Plan — E-Commerce Checkout Release

A compact, risk-based test plan in the format I use for feature releases. The example product is a web store (modeled on [SauceDemo](https://www.saucedemo.com)) releasing a redesigned cart and checkout.

I keep plans short enough that developers and product owners actually read them. The goal is shared agreement on **what we will test, what we will not, what could go wrong, and when we are done**.

---

## 1. Overview

| Item | Detail |
|---|---|
| Release | Checkout redesign v2.4 |
| Owner | QA — Matias Galvez |
| Stakeholders | Product Owner, Frontend Lead, Backend Lead, Design |
| Inputs reviewed | User stories + acceptance criteria, Figma designs, API contract (OpenAPI), previous production incidents |

## 2. Scope

### In scope
- Login (all supported user roles)
- Inventory listing, sorting, product details
- Add / remove products, cart badge and cart page
- Checkout step one (customer information + validation)
- Checkout step two (order overview, item total, tax, total)
- Order confirmation and return to inventory
- Logout and session handling
- API endpoints behind cart and checkout
- Responsive layout: desktop, tablet, mobile viewports

### Out of scope (and why)
| Area | Reason |
|---|---|
| Payment provider internals | Owned and certified by the provider; we test our integration and error handling only |
| Load testing at peak traffic | Covered by the separate k6 performance plan |
| Admin back office | No changes in this release |

## 3. Risk Assessment

I rank risk as **impact × likelihood** and use it to decide depth of testing and what gets automated first.

| # | Risk | Impact | Likelihood | Mitigation |
|---|---|---|---|---|
| R1 | Customer cannot complete checkout | High | Medium | Smoke-gated E2E on every PR; manual exploratory session per build |
| R2 | Wrong total / tax calculation | High | Medium | API tests on totals with boundary prices; manual cross-check in overview |
| R3 | Validation only on client side | High | Low | API negative tests submitting incomplete orders |
| R4 | Role-specific defects (locked, restricted users) | Medium | Medium | Data-driven login and checkout tests per role |
| R5 | Layout breaks on mobile | Medium | High | Viewport checks vs Figma at 375 / 768 / 1280 px |
| R6 | Slow login or cart under load | Medium | Low | k6 thresholds on auth and cart endpoints |

## 4. Test Approach

| Level | What | How |
|---|---|---|
| API | Status codes, schemas, totals, auth, negative cases | Postman collection + automated API suite in CI |
| UI E2E | Critical journeys (login → cart → checkout → confirmation) | Playwright, smoke on PR, full regression nightly |
| Manual exploratory | New UI, edge cases, usability, copy | Time-boxed charters (see `exploratory-testing-charter.md`) |
| Visual / responsive | Match to Figma, breakpoints | Manual on emulated devices + BrowserStack real devices |
| Accessibility | Keyboard navigation, labels, contrast | axe scan + manual keyboard pass |
| Regression | Existing behavior unchanged | Risk-based selection (see `regression-strategy.md`) |

## 5. Environments and Data

| Env | Purpose | Data |
|---|---|---|
| QA | Feature testing per build | Seeded test users per role, fixed product catalog |
| Staging | Release candidate, regression, UAT | Production-like data, masked |
| Production | Post-release smoke only | Dedicated synthetic test account |

Browsers: Chrome, Firefox, Safari (latest), plus Safari iOS and Chrome Android on real devices.

## 6. Entry Criteria
- Stories have acceptance criteria reviewed by QA during refinement.
- Build deployed to QA with release notes.
- API contract published; test data seeded.
- Unit tests and PR smoke suite green.

## 7. Exit Criteria
- 100% of planned critical (P1) test cases executed and passed.
- No open Blocker or Critical defects; Major defects have an agreed plan.
- Full automated regression green on staging (known flaky tests triaged, not ignored).
- Exploratory sessions completed for every in-scope area.
- Release readiness checklist signed off (see `regression-strategy.md`).

## 8. Sample Test Cases

| ID | Title | Priority | Type |
|---|---|---|---|
| TC-01 | Standard user completes checkout with one item | P1 | E2E, automated |
| TC-02 | Checkout shows required-field error for each empty field | P1 | E2E, automated |
| TC-03 | API rejects order without last name (400) | P1 | API, automated |
| TC-04 | Item total + tax = total for 1, 3 and 6 items | P1 | API + UI |
| TC-05 | Locked-out user sees the correct error and cannot log in | P2 | E2E, automated |
| TC-06 | Cart persists after page refresh and after logout/login | P2 | E2E |
| TC-07 | Removing the last item shows empty cart state | P3 | Manual |
| TC-08 | Checkout layout matches Figma at 375 px | P2 | Manual, visual |
| TC-09 | Checkout can be completed with keyboard only | P2 | Manual, a11y |

**Example of a fully written case**

```
TC-02  Checkout shows required-field error for each empty field
Preconditions: Logged in as standard_user, one item in cart, on Checkout step one
Steps / Expected:
  1. Leave all fields empty, click Continue  → "First Name is required", stays on step one
  2. Fill First Name only, click Continue     → "Last Name is required"
  3. Fill First + Last Name, click Continue   → "Postal Code is required"
  4. Fill all fields, click Continue          → Checkout: Overview is displayed
Data: First "Ana", Last "Lopez", Postal "4000"
```

## 9. Defect Management
- Tracked in Jira with the template in `bug-report-samples.md`.
- Severity reflects user/business impact; priority is agreed with the Product Owner.
- Blockers are raised in the team channel immediately, not only in Jira.
- Every fixed defect gets a retest and, where it makes sense, an automated regression test.

## 10. Deliverables
- This test plan
- Test cases in Zephyr / TestRail linked to stories
- Exploratory session notes
- Automated test reports (CI artifacts)
- Release readiness summary with open risks
