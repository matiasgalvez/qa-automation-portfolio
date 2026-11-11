# Schema Validation Examples

Checking only the status code tells you the server answered. Schema validation tells you **it answered with the contract the frontend and other services depend on**. It catches renamed fields, type changes (`"price": "29.99"` instead of `29.99`), missing required fields and unexpected nulls, the defects that break clients silently.

Examples use a booking resource from the public [Restful Booker](https://restful-booker.herokuapp.com/apidoc/index.html) API.

## 1. The JSON Schema

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Booking",
  "type": "object",
  "required": ["firstname", "lastname", "totalprice", "depositpaid", "bookingdates"],
  "additionalProperties": false,
  "properties": {
    "firstname":   { "type": "string", "minLength": 1 },
    "lastname":    { "type": "string", "minLength": 1 },
    "totalprice":  { "type": "integer", "minimum": 0 },
    "depositpaid": { "type": "boolean" },
    "bookingdates": {
      "type": "object",
      "required": ["checkin", "checkout"],
      "additionalProperties": false,
      "properties": {
        "checkin":  { "type": "string", "format": "date" },
        "checkout": { "type": "string", "format": "date" }
      }
    },
    "additionalneeds": { "type": "string" }
  }
}
```

Choices worth explaining:
- `additionalProperties: false` catches fields added by accident, including sensitive ones (`passwordHash`, `internalNotes`).
- `format: "date"` needs `ajv-formats`; without it, any string passes.
- Schemas live in the test repo under `schemas/`, versioned with the tests, or are generated from the OpenAPI spec.

## 2. Playwright + Ajv

```javascript
// tests/api/booking.schema.spec.js
const { test, expect } = require('@playwright/test');
const Ajv = require('ajv');
const addFormats = require('ajv-formats');
const bookingSchema = require('../../schemas/booking.schema.json');

const ajv = new Ajv({ allErrors: true });
addFormats(ajv);
const validateBooking = ajv.compile(bookingSchema);

test('GET /booking/:id matches the Booking schema @api', async ({ request }) => {
  const created = await request.post('/booking', {
    data: {
      firstname: 'Ana', lastname: 'Lopez', totalprice: 120, depositpaid: true,
      bookingdates: { checkin: '2026-12-01', checkout: '2026-12-05' },
    },
  });
  expect(created.status()).toBe(200);
  const { bookingid } = await created.json();

  const res = await request.get(`/booking/${bookingid}`, {
    headers: { Accept: 'application/json' },
  });
  expect(res.status()).toBe(200);

  const body = await res.json();
  const valid = validateBooking(body);
  // ajv errors are the most useful failure message — print them
  expect(valid, JSON.stringify(validateBooking.errors, null, 2)).toBe(true);
});
```

A failing run prints exactly what broke:

```
[
  { "instancePath": "/totalprice", "message": "must be integer" },
  { "instancePath": "", "message": "must NOT have additional properties",
    "params": { "additionalProperty": "internalNotes" } }
]
```

## 3. TypeScript + Zod (schema and type in one place)

```typescript
import { z } from 'zod';
import { test, expect } from '@playwright/test';

const Booking = z.object({
  firstname: z.string().min(1),
  lastname: z.string().min(1),
  totalprice: z.number().int().nonnegative(),
  depositpaid: z.boolean(),
  bookingdates: z.object({
    checkin: z.string().date(),
    checkout: z.string().date(),
  }).refine(d => d.checkout >= d.checkin, 'checkout must not be before checkin'),
  additionalneeds: z.string().optional(),
}).strict();

type Booking = z.infer<typeof Booking>;

test('booking response is valid @api', async ({ request }) => {
  const res = await request.get('/booking/1', { headers: { Accept: 'application/json' } });
  const parsed = Booking.safeParse(await res.json());
  expect(parsed.success, parsed.success ? '' : parsed.error.message).toBe(true);
});
```

Zod adds **business rules on top of types** (`checkout >= checkin`) and gives typed test data for free.

## 4. Cypress

```javascript
import Ajv from 'ajv';
import addFormats from 'ajv-formats';
import bookingSchema from '../fixtures/schemas/booking.schema.json';

const ajv = addFormats(new Ajv({ allErrors: true }));

Cypress.Commands.add('validateSchema', { prevSubject: true }, (body, schema) => {
  const validate = ajv.compile(schema);
  const valid = validate(body);
  expect(valid, JSON.stringify(validate.errors)).to.be.true;
  return body;
});

it('GET /booking/:id matches schema', () => {
  cy.request({ url: '/booking/1', headers: { Accept: 'application/json' } })
    .its('body')
    .validateSchema(bookingSchema);
});
```

## 5. Postman

```javascript
const schema = JSON.parse(pm.collectionVariables.get('bookingSchema'));
pm.test('matches Booking schema', () => pm.response.to.have.jsonSchema(schema));
```

## 6. Error Responses Have a Schema Too

Teams often validate only success responses. I also validate errors, because clients parse them:

```json
{
  "type": "object",
  "required": ["error", "message"],
  "properties": {
    "error":   { "type": "string" },
    "message": { "type": "string" },
    "field":   { "type": "string" }
  },
  "not": { "anyOf": [{ "required": ["stack"] }, { "required": ["sql"] }] }
}
```

The `not` block fails the test if a stack trace or SQL query leaks into the response.

## What Schema Validation Does Not Catch

Schema validation proves shape, not correctness. `"totalprice": 0` is a valid integer even when the real price was 120. I always pair it with assertions on the business values that matter (totals, ids, state transitions), and with backend checks for side effects.
