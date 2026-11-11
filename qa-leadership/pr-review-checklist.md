# Pull Request Review Checklist

I review pull requests daily, both test code and product code. On product PRs my focus is **testability, risk and missing coverage**. On test PRs it is **reliability and maintainability**. This is the checklist I work from.

## Reviewing Product Code (QA Perspective)

### Understand the change
- [ ] The PR links a ticket with acceptance criteria
- [ ] I can explain in one sentence what changed and who it affects
- [ ] Scope matches the ticket (no unrelated changes hiding in the diff)

### Risk
- [ ] Touches money, auth, permissions, personal data or data migrations? → deeper testing, flag in release notes
- [ ] Changes a shared component? → list every screen/service that uses it for regression
- [ ] Backwards compatible with existing clients (API contract, mobile apps in the field)?
- [ ] Behind a feature flag if risky; flag default is safe

### Testability
- [ ] New UI elements have `data-test` attributes
- [ ] Error states are reachable and distinguishable (not one generic message)
- [ ] Validation exists on the server, not only in the UI
- [ ] Logs include enough context (correlation id) to trace an issue
- [ ] Test data can be created through an API or seed, not only by hand

### Coverage
- [ ] Unit tests cover the new logic, including edge cases
- [ ] Existing automated tests updated if behavior changed (not just deleted)
- [ ] I note which new API / E2E tests QA will add, and link the ticket

## Reviewing Test Code

### Correctness
- [ ] The test fails when the feature is broken (would it have caught the bug it was written for?)
- [ ] Assertions check business outcomes, not just "page loaded"
- [ ] API tests assert status, schema and key values
- [ ] No assertions accidentally inside `if` blocks or swallowed by `try/catch`

### Reliability
- [ ] No hardcoded waits
- [ ] Stable selectors (`data-test`, roles), no layout-based XPath
- [ ] Independent: creates its own data, no order dependency
- [ ] Safe to run in parallel
- [ ] Passed in CI (more than once if timing-sensitive)

### Maintainability
- [ ] Follows the framework structure and naming (`automation-standards.md`)
- [ ] Page objects reused; no duplicated locators
- [ ] Test data in `test-data/` or builders, not inline magic values
- [ ] Readable title that describes the behavior
- [ ] No commented-out code, no `.only` / `test.skip` without a ticket

### Security and hygiene
- [ ] No secrets, tokens or real user data committed
- [ ] New environment variables added to `.env.example` and CI

## How I Write Review Comments

I label comments so the author knows what is blocking:

| Prefix | Meaning |
|---|---|
| **blocking:** | Must change before merge (bug, flaky pattern, missing critical coverage) |
| **suggestion:** | Better approach, author decides |
| **question:** | I need to understand something |
| **nit:** | Cosmetic, never blocks |
| **praise:** | Something done well worth repeating |

Example:

> **blocking:** `await page.waitForTimeout(3000)` will be flaky in CI. Wait for the response instead: `await page.waitForResponse('**/api/cart')`, or assert on the cart badge with `expect(...).toHaveText('1')`, which auto-waits.

> **question:** The new discount code field is validated in the UI. Does `POST /orders` also reject invalid codes? If not, I'd like to add an API negative test for it.

Good reviews explain *why* and offer a fix. The goal is that the same comment never needs to be written twice, which is also why recurring feedback ends up in the standards document.
