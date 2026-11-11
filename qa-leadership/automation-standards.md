# Automation Framework Standards

These are the standards I define for automation repositories I own. Their purpose is that any engineer, including a junior joining tomorrow, can read, run, debug and extend a test without asking the author.

## 1. Project Structure

```
├── tests/
│   ├── api/                 # API tests, grouped by service/resource
│   └── e2e/
│       ├── smoke/
│       └── regression/
├── pages/                   # Page objects (UI interaction only, no assertions on business rules)
├── fixtures/                # Test fixtures / custom test setup
├── test-data/               # Static data and builders
├── schemas/                 # JSON schemas for API contracts
├── utils/                   # Helpers (API clients, auth, data generators)
├── .github/workflows/       # CI
├── .env.example             # Every required variable, with safe defaults
└── README.md                # How to install, run, debug, contribute
```

## 2. Naming

| Item | Convention | Example |
|---|---|---|
| Test file | `<feature>.<type>.spec.ts` | `checkout.regression.spec.ts` |
| Describe | Feature or endpoint | `Checkout`, `POST /orders` |
| Test title | Behavior + expected result | `rejects order without last name` |
| Page object | `<Name>Page` | `CheckoutPage` |
| Tags | `@smoke`, `@regression`, `@api`, `@p1`, `@quarantine` | |

A test title should read like a requirement. `test1` and `checkout works` are not acceptable.

## 3. Selectors

Order of preference:
1. Dedicated test attributes: `data-test`, `data-testid`
2. Accessible roles and labels: `getByRole('button', { name: 'Checkout' })`
3. Stable ids
4. **Never:** generated classes, XPath based on layout, `nth-child` chains, text that changes with copy edits

If an element has no stable selector, I ask the developer to add a `data-test` attribute in the same PR rather than writing a fragile locator.

## 4. Waiting

- **No hardcoded waits** (`sleep`, `waitForTimeout`, `cy.wait(5000)`) except in a documented exceptional case.
- Rely on auto-waiting assertions (`await expect(locator).toBeVisible()`, `.should('be.visible')`).
- Wait for the condition that matters (a response, a state change), not for time.

## 5. Test Independence and Data

- Every test runs alone and in any order.
- Tests create their own data through the API, not through UI steps.
- Tests clean up what they create, or use data that is safe to leave.
- Unique data per worker for parallel runs (e.g., `user+${workerIndex}-${Date.now()}@test.com`).
- No real customer data, no secrets in the repo; credentials come from environment variables.

## 6. Assertions

- Each test verifies one behavior, with as many assertions as that behavior needs.
- Assert outcomes the user or business cares about, not implementation details.
- API tests assert status **and** schema **and** key values.
- Custom failure messages where the default would be unclear.
- Page objects expose actions and state; test files own the assertions about business rules.

## 7. Page Objects

```typescript
export class CheckoutPage {
  constructor(private readonly page: Page) {}

  readonly firstName = this.page.getByTestId('firstName');
  readonly lastName  = this.page.getByTestId('lastName');
  readonly postal    = this.page.getByTestId('postalCode');
  readonly error     = this.page.getByTestId('error');

  async fillCustomer({ firstName, lastName, postalCode }: Customer) {
    await this.firstName.fill(firstName);
    await this.lastName.fill(lastName);
    await this.postal.fill(postalCode);
  }

  async continue() {
    await this.page.getByTestId('continue').click();
  }
}
```

- One class per page or major component.
- Methods describe user intent (`fillCustomer`), not mechanics (`typeInField3`).
- No test logic, no conditionals that hide failures.

## 8. Reporting and Debuggability

- HTML report + JUnit XML on every run.
- Trace / screenshot / video on failure.
- API failures log request, response body and correlation id.
- Logs say what the test was doing: `Creating order for user ana+3@test.com`.

## 9. Flaky Tests

- One retry in CI maximum; pass-on-retry is reported as flaky.
- Flaky twice in a week → `@quarantine` + ticket + owner.
- Quarantined tests are fixed or deleted within the sprint. A permanent quarantine is just a deleted test that still costs CI time.

## 10. Code Review

- All test code goes through PR review, same as product code (see `pr-review-checklist.md`).
- Lint (ESLint + Prettier) runs in CI; style is not debated in reviews.
- New tests must pass in CI three times before merge when they touch timing-sensitive areas.

## 11. Definition of Done for a Test

- [ ] Covers a real requirement or risk, linked to a ticket/test case
- [ ] Follows naming, selector and waiting standards
- [ ] Independent, creates its own data, runs in parallel
- [ ] Passes locally and in CI
- [ ] Failure output is enough to debug without re-running
- [ ] Reviewed and merged

---

## Test Coverage Roadmap Template

How I plan coverage growth over a quarter, so automation work is prioritized by risk rather than by what is easiest to automate.

| Area | Business risk | Current coverage | Target | Layer | Owner | Sprint | Status |
|---|---|---|---|---|---|---|---|
| Checkout | High | 60% of P1 cases | 100% P1, 70% P2 | API + E2E | | S1–S2 | In progress |
| Login & roles | High | Happy path only | All roles, lockout, expiry | API + E2E | | S1 | Planned |
| Orders API | High | Status codes only | Schemas + negatives | API | | S2 | Planned |
| Search & filters | Medium | None | Core filters + sorting | API | | S3 | Planned |
| Account settings | Low | Manual | Smoke only | E2E | | S4 | Backlog |

Tracked every sprint:
- P1 test cases automated / total P1
- Regression duration (target: run on every merge)
- Flaky rate (target: < 2%)
- Escaped defects by area (feeds back into priority)
