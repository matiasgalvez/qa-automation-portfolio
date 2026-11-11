# Postman Collection Structure

How I organize Postman collections so they work for manual exploration *and* run unattended in CI with Newman. The example targets [Restful Booker](https://restful-booker.herokuapp.com/apidoc/index.html), a public practice API for hotel bookings.

## Collection Layout

```
Restful Booker API
├── 00 Health
│   └── GET  /ping                         → 201 Created
├── 01 Auth
│   ├── POST /auth  (valid credentials)     → saves {{token}}
│   └── POST /auth  (invalid credentials)   → "Bad credentials"
├── 02 Booking — Happy Path
│   ├── POST   /booking                     → saves {{bookingId}}
│   ├── GET    /booking/{{bookingId}}
│   ├── PUT    /booking/{{bookingId}}       (token)
│   ├── PATCH  /booking/{{bookingId}}       (token)
│   ├── DELETE /booking/{{bookingId}}       (token) → 201
│   └── GET    /booking/{{bookingId}}       → 404 after delete
├── 03 Booking — Negative
│   ├── POST   /booking  missing firstname
│   ├── POST   /booking  checkout before checkin
│   ├── PUT    /booking/{{bookingId}}  no token        → 403
│   ├── DELETE /booking/{{bookingId}}  invalid token   → 403
│   └── GET    /booking/999999999                      → 404
└── 04 Search / Filters
    ├── GET /booking?firstname=...
    └── GET /booking?checkin=...&checkout=...
```

Rules I follow:
- Folders run in order and each request reads only variables set by earlier requests in the same run.
- Every request has tests. A request without assertions is not a test.
- Negative cases live in their own folder so the happy path stays readable.

## Environments

| Variable | QA | Staging |
|---|---|---|
| `baseUrl` | `https://restful-booker.herokuapp.com` | `https://staging.example.com` |
| `username` | `admin` | from secret |
| `password` | from secret | from secret |
| `token` | set at runtime | set at runtime |
| `bookingId` | set at runtime | set at runtime |

Secrets are never committed; locally they live in the Postman vault, and in CI they are passed with `--env-var`.

## Pre-request Script — Get a Token Once

Collection-level pre-request script so any request that needs auth has a token:

```javascript
if (!pm.environment.get('token')) {
  pm.sendRequest({
    url: pm.environment.get('baseUrl') + '/auth',
    method: 'POST',
    header: { 'Content-Type': 'application/json' },
    body: {
      mode: 'raw',
      raw: JSON.stringify({
        username: pm.environment.get('username'),
        password: pm.environment.get('password'),
      }),
    },
  }, (err, res) => {
    pm.expect(err).to.be.null;
    pm.environment.set('token', res.json().token);
  });
}
```

## Test Scripts

### POST /booking — happy path

```javascript
const body = pm.response.json();

pm.test('status is 200', () => pm.response.to.have.status(200));

pm.test('responds within 1500 ms', () => {
  pm.expect(pm.response.responseTime).to.be.below(1500);
});

pm.test('returns a numeric bookingid', () => {
  pm.expect(body.bookingid).to.be.a('number');
});

pm.test('echoes the booking that was sent', () => {
  const sent = JSON.parse(pm.request.body.raw);
  pm.expect(body.booking.firstname).to.eql(sent.firstname);
  pm.expect(body.booking.totalprice).to.eql(sent.totalprice);
  pm.expect(body.booking.bookingdates).to.eql(sent.bookingdates);
});

pm.test('matches the booking schema', () => {
  // schema stored once as a collection variable (see schema-validation-examples.md)
  const schema = JSON.parse(pm.collectionVariables.get('bookingResponseSchema'));
  pm.response.to.have.jsonSchema(schema);
});

pm.environment.set('bookingId', body.bookingid);
```

### PUT /booking — no token (negative)

```javascript
pm.test('rejects update without auth', () => pm.response.to.have.status(403));
pm.test('booking is unchanged', () => {
  pm.sendRequest(`${pm.environment.get('baseUrl')}/booking/${pm.environment.get('bookingId')}`,
    (err, res) => pm.expect(res.json().firstname).to.not.eql('Hacked'));
});
```

### Request body with dynamic data

```json
{
  "firstname": "{{$randomFirstName}}",
  "lastname": "{{$randomLastName}}",
  "totalprice": {{$randomInt}},
  "depositpaid": true,
  "bookingdates": { "checkin": "2026-12-01", "checkout": "2026-12-05" },
  "additionalneeds": "Breakfast"
}
```

## Running in CI with Newman

```bash
npx newman run restful-booker.postman_collection.json \
  -e qa.postman_environment.json \
  --env-var "password=$BOOKER_PASSWORD" \
  --reporters cli,junit,htmlextra \
  --reporter-junit-export reports/newman-junit.xml \
  --reporter-htmlextra-export reports/newman-report.html \
  --bail
```

The JUnit file feeds the pipeline's test tab; the HTML report is uploaded as a build artifact. See [`github-actions-example.yml`](../ci-cd-quality-gates/github-actions-example.yml) for the pipeline.

## Things That Make Collections Hard to Maintain (and How I Avoid Them)

| Problem | Fix |
|---|---|
| Copy-pasted assertions in every request | Shared helpers in collection-level test scripts |
| Hardcoded IDs from a manual run | IDs created by the collection itself and stored in variables |
| Tests depend on data someone else changed | Each run creates and deletes its own data |
| Only status codes asserted | Status + schema + key business fields |
| Collection drifts from the API | Regenerate request skeletons from the OpenAPI spec when it changes |
