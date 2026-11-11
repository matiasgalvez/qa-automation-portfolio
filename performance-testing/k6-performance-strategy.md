# k6 Performance Testing Strategy

In my current role I built a k6 framework for backend endpoints, so performance is checked continuously instead of once before a big launch. This document describes how I plan, write and interpret those tests.

## Goals

- Know the **baseline** response time and throughput of critical endpoints.
- Catch **performance regressions** in CI before they reach production.
- Find the **breaking point** and how the system fails (gracefully or not).
- Give the team numbers, not opinions: "p95 went from 180 ms to 420 ms after commit abc123", not "it feels slow".

## Test Types

| Type | Question it answers | Shape | When |
|---|---|---|---|
| **Smoke** | Does the script work and is the system healthy under minimal load? | 1–2 VUs, 1 min | Every nightly run |
| **Load** | Does it meet targets at expected traffic? | Ramp to normal peak, hold 10–30 min | Before releases touching critical paths |
| **Stress** | Where does it break and how? | Ramp beyond peak in steps | Quarterly / before major events |
| **Spike** | Does it survive a sudden burst (campaign, sale)? | Jump to 5–10× in seconds | Before marketing events |
| **Soak** | Leaks or degradation over time? | Normal load for 2–8 h | Before major releases |

## Defining Targets (SLOs)

Targets come from product and production data, not round numbers I invent:

| Endpoint | p95 | p99 | Error rate | Expected peak |
|---|---|---|---|---|
| `POST /auth/login` | < 500 ms | < 1 s | < 0.5% | 50 req/s |
| `GET /products` | < 300 ms | < 800 ms | < 0.1% | 200 req/s |
| `POST /orders` | < 800 ms | < 1.5 s | < 0.5% | 20 req/s |

I use **percentiles, not averages**. An average of 200 ms can hide 5% of users waiting 4 seconds.

## Example Script

```javascript
// performance/load-checkout.js
import http from 'k6/http';
import { check, group, sleep } from 'k6';

const BASE = __ENV.API_URL || 'http://localhost:3000';

export const options = {
  scenarios: {
    browse_and_buy: {
      executor: 'ramping-vus',
      stages: [
        { duration: '2m', target: 50 },   // ramp up
        { duration: '10m', target: 50 },  // hold at expected peak
        { duration: '1m', target: 0 },    // ramp down
      ],
    },
  },
  thresholds: {
    http_req_failed: ['rate<0.01'],
    'http_req_duration{name:login}':    ['p(95)<500'],
    'http_req_duration{name:products}': ['p(95)<300'],
    'http_req_duration{name:order}':    ['p(95)<800', 'p(99)<1500'],
    checks: ['rate>0.99'],
  },
};

export function setup() {
  // one token per test run; per-VU login is tested in a separate scenario
  const res = http.post(`${BASE}/auth/login`,
    JSON.stringify({ email: __ENV.PERF_USER, password: __ENV.PERF_PASSWORD }),
    { headers: { 'Content-Type': 'application/json' }, tags: { name: 'login' } });
  check(res, { 'login 200': r => r.status === 200 });
  return { token: res.json('token') };
}

export default function ({ token }) {
  const auth = { headers: { Authorization: `Bearer ${token}`, 'Content-Type': 'application/json' } };

  group('browse', () => {
    const res = http.get(`${BASE}/products`, { ...auth, tags: { name: 'products' } });
    check(res, {
      'products 200': r => r.status === 200,
      'has products': r => r.json('data.length') > 0,
    });
  });

  sleep(Math.random() * 3 + 1); // think time: real users don't click every millisecond

  group('order', () => {
    const res = http.post(`${BASE}/orders`,
      JSON.stringify({ items: [{ productId: 1, quantity: 1 }] }),
      { ...auth, tags: { name: 'order' } });
    check(res, { 'order 201': r => r.status === 201 });
  });

  sleep(1);
}
```

Design notes:
- **Thresholds make the test pass or fail**, so it can gate a pipeline.
- **Named tags** give per-endpoint metrics instead of one blended number.
- **Think time** keeps the load model realistic.
- Test data (users, products) is seeded for performance runs and isolated from functional tests.

## Reading Results

```
✓ http_req_duration{name:products}...: p(95)=212ms
✗ http_req_duration{name:order}......: p(95)=1.31s   (threshold p(95)<800)
✓ http_req_failed....................: 0.21%
```

When a threshold fails I look at, in order:
1. **When** it degraded: at a VU level (capacity) or over time (leak / queue build-up)?
2. **Which** endpoint and which requests: errors or just slow?
3. **Server side** in Datadog / Grafana: CPU, memory, DB query time, connection pool, GC, downstream calls.
4. **Compare with the last baseline** for the same commit range.

The output I share is short: what was tested, the numbers vs targets, the bottleneck evidence, and a recommendation.

## CI Integration

- **Nightly**: smoke test with thresholds against QA. A failure means the build is unhealthy.
- **On demand / pre-release**: load test against staging, triggered manually (`workflow_dispatch`).
- Results exported (`--summary-export`) and kept as artifacts for trend comparison.
- k6 can also stream to Grafana for live dashboards during long runs.

## Rules of Engagement

- **Never load-test systems we do not own** (third-party APIs, public demo sites, payment providers). Use their sandbox limits or mocks.
- Agree the time window with the team; stress tests can trigger alerts and autoscaling.
- Load-test a production-like environment; results from a tiny QA box do not predict production.

---

## Backend Performance Checklist

- [ ] Critical endpoints and their p95 / p99 / error-rate targets agreed with product
- [ ] Baseline recorded for the current release
- [ ] Load model based on production traffic (mix of endpoints, think time, peak)
- [ ] Test data seeded and isolated
- [ ] Thresholds defined in the script so the run passes or fails automatically
- [ ] Monitoring dashboards open during the run (app, DB, infrastructure)
- [ ] Results compared with the previous baseline
- [ ] Regressions ticketed with evidence (graphs, slow queries, commit range)
