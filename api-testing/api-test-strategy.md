# API Test Strategy

API tests are the best value in a test suite: they are fast, stable, and they catch business-logic defects before any UI exists. In my current role I own 5 API automation repositories, and most new coverage I add lands at this layer first.

## Objective

Validate that services behave correctly for valid and invalid input, return the agreed contract, enforce security rules, and keep data consistent across systems.

## Where API Tests Sit

```
        ┌──────────┐
        │  UI E2E  │  few — critical journeys only
       ┌┴──────────┴┐
       │    API     │  many — business rules, contracts, negatives
      ┌┴────────────┴┐
      │  Unit / int  │  most — owned by developers
      └──────────────┘
```

If a rule can be checked through the API, I do not check it again through the UI. The UI test only confirms the user can reach it.

## What I Validate

| Area | Examples |
|---|---|
| **Status codes** | 200/201 on success, 400 bad input, 401 no auth, 403 wrong role, 404 missing, 409 conflict, 422 business rule |
| **Schema / contract** | Required fields, types, formats (dates, emails, UUIDs), no unexpected nulls, enum values |
| **Response body** | Correct values for the request, calculated fields (totals, tax), sorting, filtering |
| **Headers** | `Content-Type`, caching, CORS, rate-limit headers, no sensitive server info |
| **Auth & permissions** | Missing / expired / malformed token, user A cannot read or change user B's data |
| **Error handling** | Consistent error format, useful messages, no stack traces or SQL in responses |
| **Data integrity** | What the API returns matches the database and what downstream systems receive |
| **Idempotency** | Retrying a PUT or a payment request does not duplicate side effects |
| **Pagination** | First/last page, page size limits, stable ordering, total counts |
| **Performance (light)** | Response time assertions on critical endpoints; deeper testing in k6 |

## Test Design

For each endpoint I cover:

1. **Happy path** with realistic data.
2. **Each required field missing**, one at a time.
3. **Boundary values** (min, max, max+1, empty, very long strings, zero, negative).
4. **Wrong types** (string for number, array for object, null).
5. **Auth matrix** (anonymous, valid user, other user, admin, expired token).
6. **State-dependent cases** (update a deleted resource, double submit, out-of-order calls).

### Negative Test Case Examples

Example service: an order API behind a checkout.

| ID | Request | Expected |
|---|---|---|
| API-N01 | `POST /orders` without `lastName` | `400`, error names `lastName` |
| API-N02 | `POST /orders` with `quantity: 0` | `422`, quantity must be ≥ 1 |
| API-N03 | `POST /orders` with `quantity: "two"` | `400`, type error |
| API-N04 | `POST /orders` with a product id that does not exist | `404` or `422`, no order created |
| API-N05 | `POST /orders` with no token | `401` |
| API-N06 | `GET /orders/{id}` for another user's order | `403` or `404`, no data leaked |
| API-N07 | `POST /orders` with a 10,000-character `firstName` | `400`, no 500 |
| API-N08 | `POST /orders` with `"price": 0.01` overriding the catalog | Server ignores client price, uses catalog price |
| API-N09 | Same `POST /payments` sent twice with one idempotency key | One charge only |
| API-N10 | `PUT /orders/{id}` after the order is shipped | `409`, order unchanged |

API-N01 is exactly the kind of test that catches a client-side-only validation bug, like BUG-01 in [`bug-report-samples.md`](../manual-qa/bug-report-samples.md).

## Backend Validation Checklist

When a UI or API test passes, I still verify the side effects for critical flows:

- [ ] Record created/updated in the database with the right values (SQL query on SQL Server / PostgreSQL)
- [ ] No duplicate rows after retries
- [ ] Audit / history fields populated (created_by, timestamps in UTC)
- [ ] Events or messages published to downstream services
- [ ] Logs show the request with a correlation id and no unexpected errors (Kibana / Datadog)
- [ ] Metrics and dashboards reflect the action (Grafana / Datadog)
- [ ] Sensitive data is masked in logs

## Tooling

| Need | Tool |
|---|---|
| Exploration and manual checks | Postman, Swagger UI |
| Automated suites | Playwright `request`, Cypress `cy.request`, Mocha + Supertest/axios |
| Schema validation | Ajv (JSON Schema), Zod, Postman `jsonSchema` |
| Contract source | OpenAPI / Swagger spec |
| CI execution | GitHub Actions, Jenkins, Newman for Postman collections |
| Database checks | SQL Server Management Studio, `psql` |
| Logs and metrics | Datadog, Grafana, Kibana / Elastic |

See [`postman-collection-summary.md`](postman-collection-summary.md) for collection structure and [`schema-validation-examples.md`](schema-validation-examples.md) for code.

## CI/CD Usage

- API smoke suite on every PR (under 3 minutes).
- Full API regression on every merge to main and nightly.
- Tests run against an ephemeral or dedicated QA environment with seeded data.
- JUnit results published to the pipeline; failures show request, response and correlation id.

## Definition of Done for an API Test

- Covers the success case and the meaningful negative cases.
- Validates the schema, not only the status code.
- Creates and cleans up its own data; does not depend on test order.
- Runs locally and in CI with configuration from environment variables.
- Failure output includes the request, the response body and enough context to debug without re-running.
