# Page Object Model

## Purpose

The Page Object Model is used to keep UI automation maintainable, readable and scalable.

Instead of writing selectors and UI actions directly inside test files, page-specific selectors and methods are grouped into page classes or page modules.

This makes the framework easier to update when the UI changes.

## Benefits

- Reduces duplicated selectors.
- Improves test readability.
- Separates test intention from implementation details.
- Makes maintenance easier when UI elements change.
- Encourages reusable actions.
- Helps scale the framework across multiple flows and teams.

## Example Structure

```text
tests/
├── features/
│   └── checkout.feature
├── step-definitions/
│   └── checkout.steps.js
├── pages/
│   ├── login.page.js
│   ├── inventory.page.js
│   ├── cart.page.js
│   └── checkout.page.js
└── utils/
    └── test-data.js
```


## Example Page Object

```js
class LoginPage {
  get usernameInput() {
    return $('#user-name');
  }

  get passwordInput() {
    return $('#password');
  }

  get loginButton() {
    return $('#login-button');
  }

  async open() {
    await browser.url('/');
  }

  async login(username, password) {
    await this.usernameInput.setValue(username);
    await this.passwordInput.setValue(password);
    await this.loginButton.click();
  }
}

module.exports = new LoginPage();
```

## Example Test Usage

```js
const LoginPage = require('../pages/login.page');

describe('Login', () => {
  it('should allow a valid user to log in', async () => {
    await LoginPage.open();
    await LoginPage.login('standard_user', 'secret_sauce');

    await expect(browser).toHaveUrlContaining('/inventory.html');
  });
});
```

## Good Practices

- Do not place assertions inside page objects unless there is a strong reason.
- Keep page methods focused on actions.
- Use descriptive method names.
- Avoid exposing selectors directly to test files.
- Keep selectors stable and readable.
- Avoid duplicating common actions across pages.
- Use helper functions for shared behavior.

