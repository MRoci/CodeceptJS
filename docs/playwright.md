---
permalink: /playwright
title: Testing with Playwright
---

# Testing with Playwright <Badge text="Since 2.5" type="warning"/>

Playwright is a Node library to automate the [Chromium](https://www.chromium.org/Home), [WebKit](https://webkit.org/) and [Firefox](https://www.mozilla.org/en-US/firefox/new/) browsers with a single API. It enables **cross-browser** web automation that is **ever-green**, **capable**, **reliable** and **fast**.

Playwright was built similarly to [Puppeteer](https://github.com/puppeteer/puppeteer), using its API and so is very different in usage. However, Playwright has cross browser support with better design for test automaiton.

Take a look at a sample test:

```js
I.amOnPage('https://github.com');
I.click('Sign in', '//html/body/div[1]/header');
I.see('Sign in to GitHub', 'h1');
I.fillField('Username or email address', 'something@totest.com');
I.fillField('Password', '123456');
I.click('Sign in');
I.see('Incorrect username or password.', '.flash-error');
```

It's readable and simple and working using Playwright API!

## Setup

To start you need CodeceptJS with Playwright packages installed

```bash
npm install codeceptjs playwright@^0.12.1 --save
```

Or see [alternative installation options](http://codecept.io/installation/)

> If you already have CodeceptJS project, just install `playwright` package and enable a helper it in config.

And a basic project initialized

```sh
npx codeceptjs init
```

You will be asked for a Helper to use, you should select Playwright and provide url of a website you are testing.

## Configuring

Make sure `Playwright` helper is enabled in `codecept.conf.js` config:

```js
{ // ..
  helpers: {
    Playwright: {
      url: "http://localhost",
      show: true,
      browser: 'chromium'
    }
  }
  // ..
}
```

> Turn off the `show` option if you want to run test in headless mode.
> If you don't specify the browser here, `chromium` will be used. Possible browsers are: `chromium`, `firefox` and `webkit`

Playwright uses different strategies to detect if a page is loaded. In configuration use `waitForNavigation` option for that:

When to consider navigation succeeded, defaults to `load`. Given an array of event strings, navigation is considered to be successful after all events have been fired. Events can be either:
- `load` - consider navigation to be finished when the load event is fired.
- `domcontentloaded` - consider navigation to be finished when the DOMContentLoaded event is fired.
- `networkidle0` - consider navigation to be finished when there are no more than 0 network connections for at least 500 ms.
- `networkidle2` - consider navigation to be finished when there are no more than 2 network connections for at least 500 ms.

```js
  helpers: {
    Playwright: {
      url: "http://localhost",
      show: true,
      browser: 'chromium',
      waitForNavigation: "networkidle0"
    }
  }
```

When a test runs faster than application it is recommended to increase `waitForAction` config value.
It will wait for a small amount of time (100ms) by default after each user action is taken.

> ▶ More options are listed in [helper reference](http://codecept.io/helpers/Playwright/).

## Writing Tests

Additional CodeceptJS tests should be created with `gt` command:

```sh
npx codeceptjs gt
```

As an example we will use `ToDoMvc` app for testing.

### Actions

Tests consist with a scenario of user's action taken on a page. The most widely used ones are:

* `amOnPage` - to open a webpage (accepts relative or absolute url)
* `click` - to locate a button or link and click on it
* `fillField` - to enter a text inside a field
* `selectOption`, `checkOption` - to interact with a form
* `wait*` to wait for some parts of page to be fully rendered (important for testing SPA)
* `grab*` to get values from page sources
* `see`, `dontSee` - to check for a text on a page
* `seeElement`, `dontSeeElement` - to check for elements on a page

> ℹ  All actions are listed in [Playwright helper reference](http://codecept.io/helpers/Playwright/).*

All actions which interact with elements can use **[CSS or XPath locators](https://codecept.io/locators/#css-and-xpath)**. Actions like `click` or `fillField` can locate elements by their name or value on a page:

```js
// search for link or button
I.click('Login');
// locate field by its label
I.fillField('Name', 'Miles');
// we can use input name
I.fillField('user[email]','miles@davis.com');
```

You can also specify the exact locator type with strict locators:

```js
I.click({css: 'button.red'});
I.fillField({name: 'user[email]'},'miles@davis.com');
I.seeElement({xpath: '//body/header'});
```

### Interactive Pause

It's easy to start writing a test if you use [interactive pause](/basics#debug). Just open a web page and pause execution.

```js
Feature('Sample Test');

Scenario('open my website', (I) => {
  I.amOnPage('http://todomvc.com/examples/react/');
  pause();
});
```

This is just enough to run a test, open a browser, and think what to do next to write a test case.

When you execute such test with `codeceptjs run` command you may see the browser is started

```
npx codeceptjs run --steps
```

After a page is opened a full control of a browser is given to a terminal. Type in different commands such as `click`, `see`, `fillField` to write the test. A successful commands will be saved to `./output/cli-history` file and can be copied into a test.

A complete ToDo-MVC test may look like:

```js
Feature('ToDo');

Scenario('create todo item', (I) => {
  I.amOnPage('http://todomvc.com/examples/react/');
  I.dontSeeElement('.todo-count');
  I.fillField('What needs to be done?', 'Write a guide');
  I.pressKey('Enter');
  I.see('Write a guide', '.todo-list');
  I.see('1 item left', '.todo-count');
});
```

### Grabbers

If you need to get element's value inside a test you can use `grab*` methods. They should be used with `await` operator inside `async` function:

```js
const assert = require('assert');
Scenario('get value of current tasks', async (I) => {
  I.createTodo('do 1');
  I.createTodo('do 2');
  let numTodos = await I.grabTextFrom('.todo-count strong');
  assert.equal(2, numTodos);
});
```

### Within

In case some actions should be taken inside one element (a container or modal window or iframe) you can use `within` block to narrow the scope.
Please take a note that you can't use within inside another within in Playwright helper:

```js
within('.todoapp', () => {
  I.createTodo('my new item');
  I.see('1 item left', '.todo-count');
  I.click('.todo-list input.toggle');
});
I.see('0 items left', '.todo-count');
```

> [▶ Learn more about basic commands](/basics#writing-tests)

CodeceptJS allows you to implement custom actions like `I.createTodo` or use **PageObjects**. Learn how to improve your tests in [PageObjects](http://codecept.io/pageobjects/) guide.


## Extending

Playwright has a very [rich and flexible API](https://github.com/microsoft/playwright/blob/v0.12.1/docs/api.md). Sure, you can extend your test suites to use the methods listed there. CodeceptJS already prepares some objects for you and you can use them from your you helpers.

Start with creating an `MyPlaywright` helper using `generate:helper` or `gh` command:

```sh
npx codeceptjs gh
```

Then inside a Helper you can access `Playwright` helper of CodeceptJS.
Let's say you want to create `I.grabDimensionsOfCurrentPage` action. In this case you need to call `evaluate` method of `page` object

```js
// inside a MyPlaywright helper
async grabDimensionsOfCurrentPage() {
  const { page } = this.helpers.Playwright;
  await page.goto('https://www.example.com/');
  return page.evaluate(() => {
    return {
      width: document.documentElement.clientWidth,
      height: document.documentElement.clientHeight,
      deviceScaleFactor: window.devicePixelRatio
    }
  })
}
```

The same way you can also access `browser` object to implement more actions or handle events. For instance, you want to set the permissions, you can approach it with:

```js
// inside a MyPlaywright helper
async setPermissions() {
  const { browser } = this.helpers.Playwright;
  const context = browser.defaultContext()
  return context.setPermissions('https://html5demos.com', ['geolocation']);
}

> [▶ Learn more about BrowserContext](https://github.com/microsoft/playwright/blob/v0.12.1/docs/api.md#class-browsercontext)

> [▶ Learn more about Helpers](http://codecept.io/helpers/)

