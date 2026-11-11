# CI/CD Testing Strategy and Quality Gates

Automated tests only change release confidence when they run automatically, at the right moment, and their results are trusted. This is how I design pipelines so tests act as **quality gates** instead of reports nobody reads.

In my current role I built GitHub Actions pipelines that run smoke tests on every pull request and the full regression daily. Together with expanded coverage, this took regression from about a week to a few hours.

## Pipeline Design

```
 Pull request ──► Lint + unit ──► API smoke ──► UI smoke (Chromium) ──► ✅ merge allowed
                    (dev)          < 3 min        < 10 min

 Merge to main ──► Deploy to QA ──► API regression ──► UI regression ──► Deploy to staging
                                                     (3 browsers)

 Nightly ────────► Full API + UI regression ──► k6 performance smoke ──► Report + Slack

 Release ────────► Core regression on staging ──► Manual sign-off ──► Prod deploy ──► Prod smoke
```

## What Runs Where

| Trigger | Suites | Time budget | Blocks? |
|---|---|---|---|
| Pull request | Lint, unit, API smoke, UI smoke (one browser) | < 15 min | **Yes**: merge |
| Merge to main | API regression, UI regression | < 40 min | **Yes**: promotion to staging |
| Nightly (schedule) | Full regression, all browsers, k6 smoke | < 90 min | No, triaged next morning |
| Release candidate | Core regression on staging | < 30 min | **Yes**: production deploy |
| After prod deploy | Production smoke (read-only + synthetic account) | < 5 min | Triggers rollback decision |

The PR gate must be fast. If it is slow, developers stop waiting for it, and then it stops being a gate.

## Gate Rules

| Gate | Blocks on | Warns on |
|---|---|---|
| PR | Any smoke failure, lint errors | New tests skipped without a ticket |
| Promotion | Any P1 regression failure | P2/P3 failures (create ticket) |
| Performance | p95 over threshold on critical endpoints, error rate > 1% | p95 regression > 20% vs baseline |
| Accessibility | New critical axe violations | New serious violations |

Blocking on everything sounds strict but trains people to bypass the gate. Gates block on what a release cannot ship with.

## Making Failures Fast to Debug

Every run publishes, even when it passes:

- **JUnit XML** → test tab in GitHub Actions / Jenkins / CircleCI
- **HTML report** (Playwright / Mochawesome / Allure) as a build artifact
- **Traces, screenshots and videos** for failed UI tests
- **Request and response bodies** for failed API tests
- Environment, commit SHA, browser and test data in the report header

A developer should understand a failure from the report alone, without re-running it locally.

## Flaky Tests

Flaky tests destroy trust in the pipeline faster than anything else.

1. CI allows **one retry**. A test that passes on retry is marked *flaky*, not green, in the report.
2. A test that is flaky twice in a week moves to `@quarantine`: it still runs, but it no longer blocks.
3. Every quarantined test has a ticket and an owner and must be fixed or deleted within the sprint.
4. Flaky rate is a tracked metric. My target is under 2%.

Common root causes I fix: hardcoded waits, shared test data, order-dependent tests, unstable selectors, and environments that are not reset.

## Test Data and Environments

- Tests create the data they need through the API and clean up afterwards.
- Secrets come from the CI secret store, never from the repo.
- `BASE_URL` and other settings are environment variables, so the same suite runs on local, QA, staging and production smoke.
- Parallel runs use isolated data (unique emails / ids per worker).

## Visibility

- A test-results dashboard across repositories (at Forte Group I built one in React, hosted on AWS S3) shows trends rather than single runs.
- Nightly results go to the team channel with pass rate, new failures and flaky tests.
- Metrics reviewed every sprint: suite duration, pass rate, flaky rate, escaped defects.

## Example

`github-actions-example.yml` in this folder implements the PR, merge and nightly stages above for a Playwright + API + k6 project.
