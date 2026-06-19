# Cypress Automation Summary

Repository: https://github.com/matiasgalvez/cypress-js-e2e-framework

## Overview

This project demonstrates a professional Cypress Web UI automation framework using JavaScript.

The framework uses `https://the-internet.herokuapp.com` as the public demo application. This is different from the Playwright portfolio project and helps demonstrate Cypress against common UI automation patterns such as authentication, forms, dynamic loading and JavaScript dialogs.

## Tech Stack

- Cypress
- JavaScript
- Node.js
- Page Object Model
- Custom Cypress commands
- Fixtures
- Mochawesome reporting
- JUnit XML reporting
- CircleCI
- Jenkins
- dotenv

## Implemented Capabilities

- Smoke test suite
- Regression test suite
- Page Object Model structure
- Environment-based configuration
- Custom Cypress commands
- Fixture-based test data
- Screenshots on failure
- Video recording
- Mochawesome HTML/JSON reports
- JUnit XML reports for CI/CD tools
- CircleCI quality gate pipeline
- Optional Jenkins pipeline example

## Covered Flows

### Smoke Coverage

- Homepage loads successfully
- Key example links are visible
- Valid login succeeds
- Invalid login shows an error message

### Regression Coverage

- Checkbox selection and unselection
- Dropdown option selection
- Dynamic loading with hidden element
- Dynamic loading with rendered element
- JavaScript alert handling
- JavaScript confirm handling
- JavaScript prompt handling

## CI/CD Strategy

The main CI/CD example uses CircleCI.

The pipeline:

- Installs dependencies with `npm ci`
- Verifies the Cypress binary
- Runs smoke tests first
- Runs regression tests after smoke tests on `main`
- Stores JUnit test results
- Stores reports, screenshots and videos as artifacts

An optional `Jenkinsfile` is also included to show how the same framework can run in enterprise Jenkins environments.

## QA Value

This project demonstrates my ability to:

- Build Cypress frameworks from scratch.
- Structure tests using Page Object Model.
- Separate smoke and regression coverage.
- Use environment-based configuration.
- Generate CI-friendly reports.
- Configure CircleCI quality gates.
- Provide Jenkins pipeline examples.
- Validate common Web UI behaviors with maintainable automation.

## Best Practices Demonstrated

- Avoiding hardcoded waits.
- Keeping selectors and page actions inside page objects.
- Keeping test data separate from test logic.
- Using Cypress assertions and retry-ability.
- Making failures easier to debug with screenshots, videos and reports.
- Keeping tests runnable locally and in CI/CD.