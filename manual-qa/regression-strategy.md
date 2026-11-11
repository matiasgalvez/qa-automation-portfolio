# Regression Strategy

Regression testing answers one question before every release: **did we break something that used to work?** The challenge is answering it fast enough that it does not block delivery.

At Forte Group, moving regression from a mostly manual cycle to automated, CI-integrated suites took regression validation from about **one week to a few hours**. This document describes the approach behind that.

## Principles

1. **Risk decides depth.** Not every area needs the same coverage on every release.
2. **Automate the repeatable, explore the new.** Stable, high-value flows go to automation; humans focus on changed areas and edge cases.
3. **Regression runs continuously, not only at the end.** Smoke on every PR, full suite nightly, targeted suite on release candidates.
4. **A red build must mean something.** Flaky tests are fixed or quarantined, never ignored.

## Regression Tiers

| Tier | Purpose | Size | When | Gate |
|---|---|---|---|---|
| **Smoke** | Is the build usable at all? | 10–20 checks, < 10 min | Every PR, every deploy | Blocks merge / deploy |
| **Core regression** | Critical business flows + recently changed areas | ~30% of suite | Release candidate, hotfixes | Blocks release |
| **Full regression** | Everything automated, all browsers | Full suite | Nightly + before major releases | Failures triaged next morning |
| **Manual regression** | Areas that are hard or not worth automating | Checklist | Release candidate | Sign-off by QA |

## Selecting What to Run

For a given release I combine:

- **Change impact** — which services, components and pages changed (from the PRs and the release notes).
- **Business criticality** — login, checkout, payments, permissions and data changes always run.
- **Defect history** — areas with recent production incidents or frequent bugs get extra coverage.
- **Dependencies** — a change in a shared component (e.g., a form input) pulls in every screen that uses it.

Every test case is tagged (`@smoke`, `@regression`, `@checkout`, `@api`, `@p1`) so selection is a filter, not a manual exercise.

## Manual vs Automated Split

| Automate | Keep manual |
|---|---|
| Stable, repetitive, high-value flows | Brand-new features still changing |
| Data-driven checks (many users, roles, inputs) | Visual and UX judgment |
| API contracts and schemas | One-off migrations |
| Cross-browser smoke | Complex third-party flows without sandbox |
| Anything that runs every release | Exploratory testing of changed areas |

## Keeping the Suite Healthy

- **Flaky test policy:** a test that fails without a product change is moved to a quarantine tag within a day, a ticket is created, and it is fixed or deleted within the sprint.
- **Retries:** at most one retry in CI, and retried-then-passed tests are reported as flaky, not green.
- **Ownership:** each suite has an owner; failures are triaged the same morning.
- **Pruning:** tests that duplicate lower-level coverage (API or unit) are removed from the UI suite.
- **Metrics I track:** suite duration, pass rate, flaky rate, defects caught pre-release vs escaped to production.

## Hotfix Regression

For an urgent fix, full regression is too slow. I run:

1. Smoke suite on the hotfix build.
2. Targeted tests for the changed component and its direct dependencies.
3. A short manual check of the fixed scenario plus one adjacent scenario.
4. Production smoke after deploy.

---

## Release Readiness Checklist

Used for go / no-go before every production release.

### Quality
- [ ] All P1 test cases executed and passed
- [ ] Automated smoke and core regression green on staging
- [ ] Full regression run reviewed; every failure has an explanation (bug / flaky / env)
- [ ] No open Blocker or Critical defects
- [ ] Open Major defects reviewed and accepted by the Product Owner
- [ ] Exploratory sessions completed for changed areas

### Non-functional
- [ ] Performance thresholds met for changed endpoints (k6)
- [ ] Accessibility scan shows no new critical violations
- [ ] Mobile / responsive checks done for changed screens

### Release logistics
- [ ] Release notes reviewed (QA verifies they match what was tested)
- [ ] Feature flags and configuration confirmed for production
- [ ] Database migrations tested on staging, including rollback
- [ ] Rollback plan documented
- [ ] Production smoke test plan and test account ready
- [ ] Monitoring dashboards (Datadog / Grafana) and alerts identified for the release window

### Sign-off
| Role | Name | Decision |
|---|---|---|
| QA | | Go / No-go |
| Product Owner | | Go / No-go |
| Engineering Lead | | Go / No-go |
