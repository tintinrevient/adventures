---
layout: post
title: Unit and functional testing in Node.js
date: 2017-07-06 09:11:29
---

In this post I am going to write about unit testing in Node.js. I am going to use [RisingStack's Node Hero tutorial](https://blog.risingstack.com/node-hero-node-js-unit-testing-tutorial/) as a guidance. You can read the tutorial online, or you can [downlad it in a clutter-free PDF format](https://risingstack.com/resources/node-hero) as well. Another resource I recommend is this presentation by Stacy Kirk: [Full stack testing with Node.js](https://www.youtube.com/watch?v=i4Eu3Cczkek)

> “Tests are more than just safeguards - they provide a
living documentation for your codebase.” — RisingStack

We write unit tests to continously check if a given module works. We should always write tests for the exposed methods, not for the internal workings of the given module.

## Modules for Node.js unit testing

- **test runner:** [mocha](https://www.npmjs.com/package/mocha) or [tape](https://www.npmjs.com/package/tape)
- **assertion library:** [chai](http://chaijs.com/) or [assert](https://www.npmjs.com/package/assert)
- **test spies, stubs and mocks**: [sinon](http://sinonjs.org/)
- **code coverage**: [istanbul](https://github.com/gotwarlost/istanbul)

## Spies, stubs and mocks

We can use **spies** to get information about the function calls, like how many times they were called or what arguments were passed to them.

```javascript
it('calls subscribers on publish', function () {
	var callback = sinon.spy()
	PubSub.subscribe('message', callback)
	PubSub.publishSync('message')
	assertTrue(callback.called)
})
// example taken from the sinon documentation site:
// http://sinonjs.org/docs/
 ```

 **Stubs** are like spies, but they replace the target function. We can easily control a method's behaviour to force a code path, or to prevent calls to external resources.

 ```javascript
it('calls all subscribers, even if there are exceptions', function (){
	var message = 'an example message'
	var error = 'an example error message'
	var stub = sinon.stub().throws()
	var spy1 = sinon.spy()
	var spy2 = sinon.spy()

	PubSub.subscribe(message, stub)
	PubSub.subscribe(message, spy1)
	PubSub.subscribe(message, spy2)

	PubSub.publishSync(message, undefined)

	assert(spy1.called)
	assert(spy2.called)
	assert(stub.calledBefore(spy1))
})
// example taken from the sinon documentation site:
// http://sinonjs.org/docs/
 ```

A **mock** is a fake method with a pre-programmed behavior. 

```javascript
it('calls all subscribers when exceptions happen', function () {
	var myAPI = {
		method: function () {}
	}
	var spy = sinon.spy()
	var mock = sinon.mock(myAPI)
	mock.expects(“method”).once().throws()

	PubSub.subscribe(“message”, myAPI.method)
	PubSub.subscribe(“message”, spy)
	PubSub.publishSync(“message”, undefined)

	mock.verify()
	assert(spy.calledOnce)
	// example taken from the sinon documentation site:
	// http://sinonjs.org/docs/
})
```

## Setting up a testing environment using Mocha and Chai

I am going to set up a small testing environment using [this video tutorial](https://www.youtube.com/watch?v=MLTRHc5dk6s) by Traversy Media.

Let's install mocha and chai.

```bash
npm install mocha chai --save-dev
```

Inside `package.json` replace the `"test"` command to `"mocha"`.

```json
// Inside package.json
"scripts": {
	"test": "mocha",
	"start": "node dist/server.js"
}
```

Let's create a simple component which returns a text `hello`.

```javascript
// greeter.js
module.exports = function() {
	return 'hello';
}
```

Inside `/test` folder (because that's the default folder for mocha), let's create a test suite for that.

```javascript
// test/greeter.spec.js
const assert = require('chai').assert;
const app = require('../greeter');

describe('App', function(){
	it('app should return hello', function(){
		assert.equal(app(), 'hello');
	});
});
```

If we run `npm run test` we should see our test passing. If we want a cleaned test result output, we can modify the `mocha` script in our `package.json` to the following:

```json
"test": "mocha || true"
```

### Importing methods instead of entire modules

```javascript
// greeter.js
module.exports = 
{
	sayHello: function()
	{
		return "hello";
	}
}

```

```javascript
// greeter.spec.js
const assert = require('chai').assert;
const sayHello = require('../greeter').sayHello;

describe('App', function(){
	it('sayHello should return hello', function(){
		let result = sayHello();
		assert.equal(result, 'hello');
	});
});
```

### Type check

```javascript
// greeter.spec.js
it('sayHello should return type string', function(){
	let result = sayHello();
	assert.typeOf(result, 'string');
})
```

We wrote our first tests. Now let's create a convinient development environment using TypeScript.

## Project structure using TypeScript

I am a big fan of TypeScript so I am going to build an environment in which tests can be written in TS as well. 

By default, mocha will look for the test files in  the roots `/test` folder. We can change that by providing a regex expression as a parameter when running mocha in `package.json` like so:

```json
{
	"scripts": {
		"test": "mocha dist/**/**.spec.js",
		"start": "node dist/server.js"
	}
}
```

Now, we have to install the type definitions.

```bash
npm install @types/mocha @types/chai --save
```

If everything went well, we can see that the new dependencies have been added to `package.json` and to the `node_modules` directory. Our text editor might need a restart in order to pick up the new definition files. 

Now we can combile our solution using `tsc`.

## Automated builds and tests

The last thing I want to do is to watch TypeScript files on the run so it recompiles the solution automatically whenever a change is made during development, and also rerun the tests, so if a developer breakes something he/she can see it immidiately. To do so, let's install `nodemon` first, which can watch for file changes and run custom scripts whenever it detects changes. For more information visit the [official documentation](https://github.com/remy/nodemon).

```bash
npm install nodemon --save-dev
```

Now we are going to create two new scripts: `compile` and `dev`. `dev` is going to kick off nodemon, which is going to watch for changes in `.ts` files and whenever that happens it will run the `compile`, `test` and `start` scripts, which will compile the solution, run the tests and then run the app.

```json
{
	"scripts": {
		"test": "mocha dist/**/**.spec.js",
		"build": "tsc",
		"start": "node dist/server.js",
		"dev": "nodemon --watch src -e ts --exec \"npm run build && npm run test && npm start\""
	}
}
```

Our environment is ready, we can start writing tests with Mocha in TypeScript. 

## Functional tests

What is the difference between unit testing and functional testing? [Quoted from Debasis Pradhan](http://www.softwaretestingtricks.com/2007/01/unit-testing-versus-functional-tests.html): 

> Unit tests tell a developer that the code is doing things right; functional tests tell a developer that the code is doing the right things.

In our example we simply checked whether a given module behaves the way we expect it to behave. A functional test should be written from the user's perspective, let's say that when the user visit's the page, he/she will get the expected result in the browser. To do that, [based on this tutorial by François Zaninotto](http://www.redotheweb.com/2013/01/15/functional-testing-for-nodejs-using-mocha-and-zombie-js.html), we are going to create a server for the sake of testing, create a [zombie browser](http://zombie.js.org/), and navigate to the home page. 

```bash
# Install zombie
npm install --save-dev zombie
```

Because `before()` function starts the server on a custom port, it may cause confusion when it tries to execute the development version of the application. To avoid that, we are going to change `server.ts` a bit, so it only kicks off the server when it is called directly (by `node` and not by `before()`). Read more about the `module.parent` on [StackOverflow](https://stackoverflow.com/questions/13651945/what-is-the-use-of-module-parent-in-node-js-how-can-i-refer-to-the-requireing).

```ts
// server.ts

import * as http from "http";

var port: number = 5000;

function app (req: http.RequestOptions, res: http.ServerResponse): void {
    res.writeHead(200, {'Content-Type': 'text/html'});
    res.end('Hello World!');
}

module.exports = app;

// Only start the server when code is called directly
if (!module.parent) {
    http.createServer(app).listen(port, function() {
        console.log("Server listening on port " + port);
    });
}

```

Now we can write our functional test:

```ts
// functional.spec.ts

import * as http from "http";
process.env.NODE_ENV = 'test';

var app = require('../server');
var Browser:any  = require('zombie');
var assert: any = require('assert');

describe('Functional test', function() {

	var server: any, browser: any;

	before(() => {
		server = http.createServer(app).listen(5000);
		browser = new Browser({site: 'http://localhost:5000'});
	});

	before((done) => {
		browser.visit('/', done);
	});

	it('should open the page properly', () => {
		assert.ok(browser.success);
		assert.equal(browser.text('body'), 'Hello World!');
	});

	after((done) => {
		server.close(done);
	});
});
```

To make sure that our functional test works as expected, try changing the expected text `Hello World!` to something else. If we did everything right, the test should only pass when the text matches with the exact same message delivered by the server (defined in `server.ts`).