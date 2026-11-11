# Bug Report Samples

These are real defects I reproduced on [SauceDemo](https://www.saucedemo.com), a public demo store that ships intentional bugs behind specific test users. Every step below was re-run in Chromium (November 2026) before writing it up, and the evidence lines come from that run.

I write bug reports so a developer can reproduce the issue in under two minutes without asking me anything. That means one defect per ticket, exact data, the expected behavior tied to a requirement or to the baseline user, and severity justified by business impact rather than by how annoying it looks.

## Template I Use

```
Title:        [Area] Short, specific symptom (who / where / what)
Environment:  URL, build/version, browser + version, OS, device/viewport
User / Data:  Account, role, test data used
Preconditions:
Steps to Reproduce:
  1.
Expected Result:   (cite the requirement, AC or baseline behavior)
Actual Result:
Evidence:          Screenshot / video / HAR / console log / request + response
Severity:          Blocker | Critical | Major | Minor | Trivial   (impact)
Priority:          P1–P4                                          (urgency)
Frequency:         Always | Intermittent (x of y) | Once
Notes:             Scope checked, workaround, suspected layer (UI / API / data)
```

---

## BUG-01 — Checkout accepts the form with an empty Last Name (validation bypass)

| Field | Value |
|---|---|
| Environment | https://www.saucedemo.com · Chromium 131 · macOS · 1280×720 |
| User | `error_user` / `secret_sauce` |
| Severity / Priority | **Critical / P1** — order data is incomplete; downstream fulfillment and invoicing would fail |
| Frequency | Always (5/5) |

**Steps to Reproduce**
1. Log in as `error_user`.
2. Add *Sauce Labs Backpack* to the cart and open the cart.
3. Click **Checkout**.
4. Type `Ana` in First Name, `Lopez` in Last Name, `4000` in Postal Code.
5. Click **Continue**.

**Expected Result**
Last Name keeps the typed value. If it is empty, the form shows `Error: Last Name is required` and stays on step one (this is what `standard_user` gets).

**Actual Result**
The Last Name field never keeps its value — it is empty after typing. The form still advances to `/checkout-step-two.html`, so the order continues without a last name.

**Evidence**
`inputValue('[data-test="lastName"]') === ""` after typing, followed by a successful navigation to step two.

**Notes**
Two defects in one flow: the input does not bind, *and* client-side validation is skipped for this user. I would file them separately and link them, because the validation bypass is the more dangerous one and should also be checked at the API layer (the server must reject an order without a last name regardless of the UI).

---

## BUG-02 — "Finish" does nothing and throws a JavaScript error; order cannot be completed

| Field | Value |
|---|---|
| User | `error_user` |
| Severity / Priority | **Blocker / P1** — revenue path is broken |
| Frequency | Always |

**Steps to Reproduce**
1. Complete BUG-01 steps 1–5 to reach *Checkout: Overview*.
2. Click **Finish**.

**Expected Result**
Navigate to `/checkout-complete.html` with "Thank you for your order!".

**Actual Result**
The page stays on `/checkout-step-two.html`. No message is shown to the user.

**Evidence (console)**
```
Uncaught TypeError: Cannot read properties of undefined (reading 'value')
Uncaught TypeError: wa.cesetRart is not a function
```

**Notes**
Silent failure is part of the bug: even if the backend call failed, the user should see an error and a retry option.

---

## BUG-03 — Last Name input writes into the First Name field

| Field | Value |
|---|---|
| User | `problem_user` |
| Severity / Priority | **Critical / P1** — checkout cannot be completed |

**Steps to Reproduce**
1. Log in as `problem_user`, add any product, go to **Checkout**.
2. Type `Ana` in First Name.
3. Click Last Name and type `Lopez`.
4. Type `4000` in Postal Code and click **Continue**.

**Expected Result**
First Name = `Ana`, Last Name = `Lopez`; the form advances.

**Actual Result**
First Name = `Lopez`, Last Name = empty. The form shows `Error: Last Name is required` and the user is stuck.

**Notes**
Likely the Last Name input's change handler is bound to the First Name model. Check other forms that reuse the same input component.

---

## BUG-04 — "Add to cart" silently fails for 3 of 6 products

| Field | Value |
|---|---|
| User | `problem_user` |
| Severity / Priority | **Major / P2** — half the catalog cannot be purchased |

**Steps to Reproduce**
1. Log in as `problem_user`.
2. Click **Add to cart** on all six products on the inventory page.

**Expected Result**
Every button switches to **Remove** and the cart badge shows `6`.

**Actual Result**
The badge shows `3`. These buttons never switch to Remove and the items are not added:
- Sauce Labs Bolt T-Shirt
- Sauce Labs Fleece Jacket
- Test.allTheThings() T-Shirt (Red)

---

## BUG-05 — Product name opens the wrong product's detail page

| Field | Value |
|---|---|
| User | `problem_user` |
| Severity / Priority | **Major / P2** — user may buy the wrong item |

**Steps to Reproduce**
1. Log in as `problem_user`.
2. Click the product name **Sauce Labs Backpack**.

**Expected Result**
Details page for *Sauce Labs Backpack* (`inventory-item.html?id=4`).

**Actual Result**
Details page for *Sauce Labs Fleece Jacket* (`inventory-item.html?id=5`).

**Notes**
Off-by-one in the item id mapping is the first thing I would check.

---

## BUG-06 — Login takes ~5 seconds for one account (≈125× slower than baseline)

| Field | Value |
|---|---|
| User | `performance_glitch_user` vs `standard_user` |
| Severity / Priority | **Major / P2** — perceived outage; users retry and create duplicate sessions |
| Frequency | Always (measured on 3 runs) |

**Steps to Reproduce**
1. Log in as `standard_user` and measure time from clicking **Login** to the inventory page rendering.
2. Repeat with `performance_glitch_user`.

**Expected Result**
Comparable timings; product target is under 1 second.

**Actual Result**

| User | Click → inventory loaded |
|---|---|
| `standard_user` | ~40 ms |
| `performance_glitch_user` | ~5,040 ms |

**Notes**
Because only one account is affected, I would check account-specific work on login (profile loading, feature flags, a sleep in a code path) before suspecting infrastructure. A k6 check on the auth endpoint with a `p(95)<1000` threshold would catch a regression like this in CI.

---

## BUG-07 — Prices on the inventory page differ from the catalog price

| Field | Value |
|---|---|
| User | `visual_user` vs `standard_user` |
| Severity / Priority | **Critical / P1** — displayed price does not match the price charged |

**Steps to Reproduce**
1. Log in as `standard_user` and note the six prices.
2. Log out, log in as `visual_user`, compare.

**Actual Result**

| Product | standard_user | visual_user |
|---|---|---|
| Sauce Labs Backpack | $29.99 | $72.72 |
| Sauce Labs Bike Light | $9.99 | $34.84 |
| Sauce Labs Bolt T-Shirt | $15.99 | $30.58 |

**Notes**
Pricing defects are legal and trust issues, not cosmetic ones. I would also verify which price reaches the cart, the overview total and the API order payload.

---

## BUG-08 — Sorting is broken and shows a raw error dialog

| Field | Value |
|---|---|
| Users | `problem_user`, `error_user` |
| Severity / Priority | **Minor / P3** (`problem_user`), **Major / P2** (`error_user`, because of the alert) |

**Steps to Reproduce**
1. Log in as `problem_user`. Choose **Name (Z to A)**.
2. Log in as `error_user`. Choose **Name (Z to A)**.

**Expected Result**
Products re-order Z→A, first item is *Test.allTheThings() T-Shirt (Red)*.

**Actual Result**
- `problem_user`: order does not change; *Sauce Labs Backpack* stays first.
- `error_user`: a browser alert appears: *"Sorting is broken! This error has been reported to Backtrace."*

**Notes**
A native `alert()` with internal tooling names should never reach end users. Even when a feature fails, the UI should degrade gracefully.

---

## Why These Examples

They show the things I look for beyond "the button is broken": validation that only exists on the client, silent failures, defects that only appear for one role or account, data integrity (price, order fields), and performance outliers measured against a baseline instead of described as "slow".
