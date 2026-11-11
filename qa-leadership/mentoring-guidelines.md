# QA Mentoring Guidelines

I mentor junior QA engineers on automation usage, framework maintenance, coverage expansion and troubleshooting. These guidelines describe how I structure it. The goal is independence: a mentee who can own a feature's quality end to end without me.

## Principles

- **Pair first, review second, step back third.** Show, then do it together, then let them lead with a safety net.
- **Explain the why.** A rule without a reason gets broken the first time it is inconvenient.
- **Real work over exercises.** Mentees learn on real tickets with real impact, sized to their level.
- **Safe to ask, safe to fail.** No question is too basic; mistakes caught in review are cheap learning.
- **Feedback is specific and regular**, not saved for performance reviews.

## Onboarding Plan (first 90 days)

### Days 1–30: Understand and run
- Product walkthrough: users, critical flows, how the business makes money
- Architecture overview: services, environments, where logs and dashboards live
- Run every automation suite locally; read and explain three existing tests
- Pair on exploratory testing of a live feature
- First contribution: fix a small flaky test or add a missing assertion, with me pairing
- **Outcome:** can run, read and debug the existing suites

### Days 31–60: Contribute with support
- Write test cases for a story from its acceptance criteria; I review before execution
- Automate 3–5 test cases for a feature (API first, then UI), via PR with my review
- Join refinement sessions and ask at least one testability question per story
- Triage nightly failures together: product bug, test bug, or environment?
- **Outcome:** delivers tested features with review

### Days 61–90: Own
- Owns QA for a small feature: test plan, manual testing, automation, release sign-off
- Reviews a teammate's test PR using the checklist
- Presents one improvement to the team (a flaky fix, a new helper, a coverage gap)
- **Outcome:** works independently on normal-sized features

## Skill Areas I Cover

| Area | Topics |
|---|---|
| Testing fundamentals | Test design techniques (boundaries, equivalence classes, decision tables), risk-based prioritization, exploratory charters |
| Bug reporting | Reproducible steps, evidence, severity vs priority, see [`bug-report-samples.md`](../manual-qa/bug-report-samples.md) |
| Automation | Framework structure, page objects, selectors, waiting, fixtures, data |
| API | HTTP basics, status codes, auth, Postman, schema validation |
| Debugging | Reading traces, DevTools network tab, logs in Kibana/Datadog, isolating a flaky test |
| CI/CD | How pipelines run tests, reading reports, quality gates |
| Git | Branching, small PRs, writing good PR descriptions, responding to review |
| Collaboration | Asking good questions in refinement, giving and receiving review feedback |

## Weekly Rhythm

| Activity | Frequency | Purpose |
|---|---|---|
| 1:1 | Weekly, 30 min | Blockers, goals, feedback both ways |
| Pairing | 2–3 sessions per week early on, then on demand | Learn by doing |
| PR reviews | Every PR | Teach standards in context |
| Failure triage | Daily, together at first | Learn debugging and the system |

## Giving Feedback

- **Specific:** "The assertion only checks the page loaded; it would pass even if the total was wrong. Assert on the total." Not "make the test better".
- **Timely:** in the PR or right after the session, not weeks later.
- **Balanced:** call out what was done well so it gets repeated.
- **Actionable:** every piece of critical feedback includes a next step or an example.

## Signs Mentoring Is Working

- Mentee's PRs need fewer review rounds over time
- They find and report bugs in areas nobody pointed them to
- They ask "how should we test this?" during refinement, before code exists
- They start reviewing others' tests and teaching what they learned
- Recurring review comments drop because the lessons made it into the standards
