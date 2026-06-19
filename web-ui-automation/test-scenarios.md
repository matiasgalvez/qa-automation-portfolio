# Web UI Test Scenarios

## Application Type

Example application: E-commerce web application.

## Scope

This document includes sample Web UI test scenarios for smoke, regression and exploratory testing.
The goal is to demonstrate how I organize UI test coverage by feature, risk level and business value.

---

## Login Scenarios

### UI-LOGIN-001 — Valid Login

**Type:** Smoke
**Priority:** High

**Scenario:**
User can log in with valid credentials.

**Expected Result:**
The user is redirected to the product inventory or dashboard page.

---

### UI-LOGIN-002 — Invalid Password

**Type:** Regression
**Priority:** High

**Scenario:**
User cannot log in with an invalid password.

**Expected Result:**
An error message is displayed and the user remains on the login page.

---

### UI-LOGIN-003 — Empty Username

**Type:** Regression
**Priority:** Medium

**Scenario:**
User attempts to log in without entering a username.

**Expected Result:**
A required field validation message is displayed.

---

### UI-LOGIN-004 — Empty Password

**Type:** Regression
**Priority:** Medium

**Scenario:**
User attempts to log in without entering a password.

**Expected Result:**
A required field validation message is displayed.

---

### UI-LOGIN-005 — Logout

**Type:** Smoke
**Priority:** High

**Scenario:**
Logged-in user can log out successfully.

**Expected Result:**
The user session ends and the login page is displayed.

---

## Product List Scenarios

### UI-PRODUCT-001 — Product List Loads

**Type:** Smoke
**Priority:** High

**Scenario:**
Product list loads successfully after login.

**Expected Result:**
Products are displayed with name, image and price.

---

### UI-PRODUCT-002 — Product Details Are Displayed

**Type:** Regression
**Priority:** High

**Scenario:**
User can view product information from the product list.

**Expected Result:**
The product name, description, image and price are visible and correct.

---

### UI-PRODUCT-003 — Sort Products by Price Low to High

**Type:** Regression
**Priority:** Medium

**Scenario:**
User sorts products from lowest price to highest price.

**Expected Result:**
Products are displayed in ascending price order.

---

### UI-PRODUCT-004 — Sort Products by Price High to Low

**Type:** Regression
**Priority:** Medium

**Scenario:**
User sorts products from highest price to lowest price.

**Expected Result:**
Products are displayed in descending price order.

---

## Cart Scenarios

### UI-CART-001 — Add Product to Cart

**Type:** Smoke
**Priority:** High

**Scenario:**
User adds a product to the cart.

**Expected Result:**
The product is added and the cart badge updates correctly.

---

### UI-CART-002 — Remove Product from Cart

**Type:** Regression
**Priority:** High

**Scenario:**
User removes a product from the cart.

**Expected Result:**
The product is removed and the cart total updates correctly.

---

### UI-CART-003 — Validate Cart Product Information

**Type:** Regression
**Priority:** High

**Scenario:**
User opens the cart after adding a product.

**Expected Result:**
The cart displays the correct product name, quantity and price.

---

### UI-CART-004 — Continue Shopping

**Type:** Regression
**Priority:** Low

**Scenario:**
User clicks the continue shopping button from the cart page.

**Expected Result:**
The user is redirected back to the product list.

---

## Checkout Scenarios

### UI-CHECKOUT-001 — Complete Checkout with Valid Data

**Type:** Smoke
**Priority:** Critical

**Scenario:**
User completes checkout using valid customer information.

**Expected Result:**
The order is completed and the confirmation page is displayed.

---

### UI-CHECKOUT-002 — Required Field Validation

**Type:** Regression
**Priority:** High

**Scenario:**
User attempts to continue checkout without completing required fields.

**Expected Result:**
The system displays validation messages for missing required fields.

---

### UI-CHECKOUT-003 — Cancel Checkout

**Type:** Regression
**Priority:** Medium

**Scenario:**
User cancels checkout before completing the purchase.

**Expected Result:**
The user is redirected back to the cart or product page, depending on the application behavior.

---

### UI-CHECKOUT-004 — Validate Checkout Overview

**Type:** Regression
**Priority:** High

**Scenario:**
User reaches the checkout overview page.

**Expected Result:**
The item total, taxes, final price and product information are displayed correctly.

---

### UI-CHECKOUT-005 — Order Confirmation

**Type:** Smoke
**Priority:** Critical

**Scenario:**
User finishes the checkout process.

**Expected Result:**
A confirmation message is displayed and the order is completed successfully.

---

## Negative Scenarios

### UI-NEG-001 — Invalid Login Error

**Type:** Regression
**Priority:** High

**Scenario:**
User enters invalid login credentials.

**Expected Result:**
The system displays an authentication error message.

---

### UI-NEG-002 — Missing First Name During Checkout

**Type:** Regression
**Priority:** High

**Scenario:**
User attempts to continue checkout without entering a first name.

**Expected Result:**
The system displays a first name required validation message.

---

### UI-NEG-003 — Missing Last Name During Checkout

**Type:** Regression
**Priority:** High

**Scenario:**
User attempts to continue checkout without entering a last name.

**Expected Result:**
The system displays a last name required validation message.

---

### UI-NEG-004 — Missing Postal Code During Checkout

**Type:** Regression
**Priority:** High

**Scenario:**
User attempts to continue checkout without entering a postal code.

**Expected Result:**
The system displays a postal code required validation message.

---

## Exploratory Testing Ideas

* Validate behavior using keyboard navigation.
* Validate browser back and forward behavior.
* Validate responsive behavior on desktop, tablet and mobile viewports.
* Validate error messages, wording and consistency.
* Validate loading states and empty states.
* Validate visual consistency against Figma designs.
* Validate behavior after refreshing the page.
* Validate session behavior after logout.
* Validate product totals after adding and removing items.
* Validate checkout behavior with interrupted or incomplete flows.

---

## Prioritization Criteria

When deciding what to automate first, I prioritize scenarios that are:

* Business-critical.
* Frequently executed during regression.
* High-risk during releases.
* Related to authentication, cart, checkout or payments.
* Stable enough for automation.
* Valuable as smoke checks in CI/CD pipelines.
