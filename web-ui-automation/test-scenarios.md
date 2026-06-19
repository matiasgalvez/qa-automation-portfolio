```md
# Web UI Test Scenarios

## Application Type

Example application: E-commerce web application.

## Scope

This document includes sample Web UI test scenarios for smoke, regression and exploratory testing.

## Login Scenarios

| ID | Scenario | Type | Priority |
|---|---|---|---|
| UI-LOGIN-001 | User can log in with valid credentials | Smoke | High |
| UI-LOGIN-002 | User cannot log in with invalid password | Regression | High |
| UI-LOGIN-003 | User cannot log in with empty username | Regression | Medium |
| UI-LOGIN-004 | User cannot log in with empty password | Regression | Medium |
| UI-LOGIN-005 | Error message is displayed for invalid credentials | Regression | High |
| UI-LOGIN-006 | User can log out successfully | Smoke | High |

## Product List Scenarios

| ID | Scenario | Type | Priority |
|---|---|---|---|
| UI-PRODUCT-001 | Product list loads successfully | Smoke | High |
| UI-PRODUCT-002 | Product name, image and price are displayed | Regression | High |
| UI-PRODUCT-003 | User can sort products by price low to high | Regression | Medium |
| UI-PRODUCT-004 | User can sort products by price high to low | Regression | Medium |
| UI-PRODUCT-005 | Product detail page opens correctly | Regression | Medium |

## Cart Scenarios

| ID | Scenario | Type | Priority |
|---|---|---|---|
| UI-CART-001 | User can add a product to the cart | Smoke | High |
| UI-CART-002 | Cart badge updates after adding a product | Regression | High |
| UI-CART-003 | User can remove a product from the cart | Regression | High |
| UI-CART-004 | Cart displays correct product information | Regression | High |
| UI-CART-005 | User can continue shopping from cart page | Regression | Low |

## Checkout Scenarios

| ID | Scenario | Type | Priority |
|---|---|---|---|
| UI-CHECKOUT-001 | User can complete checkout with valid data | Smoke | Critical |
| UI-CHECKOUT-002 | Required field validation is displayed | Regression | High |
| UI-CHECKOUT-003 | User can cancel checkout | Regression | Medium |
| UI-CHECKOUT-004 | Checkout overview displays correct item total | Regression | High |
| UI-CHECKOUT-005 | Order confirmation page is displayed | Smoke | Critical |

## Negative Scenarios

| ID | Scenario | Type | Priority |
|---|---|---|---|
| UI-NEG-001 | Invalid login shows an error message | Regression | High |
| UI-NEG-002 | Checkout cannot continue with missing first name | Regression | High |
| UI-NEG-003 | Checkout cannot continue with missing last name | Regression | High |
| UI-NEG-004 | Checkout cannot continue with missing postal code | Regression | High |

## Exploratory Testing Ideas

- Validate behavior using keyboard navigation.
- Validate browser back and forward behavior.
- Validate responsive behavior on different viewport sizes.
- Validate error messages and wording.
- Validate loading states.
- Validate visual consistency with designs.
- Validate behavior after refreshing the page.
- Validate session behavior after logout.