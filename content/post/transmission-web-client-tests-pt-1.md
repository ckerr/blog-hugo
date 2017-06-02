+++
banner = "banners/backlit-keyboard.jpg"
categories = ["development"]
date = "2017-05-31T19:21:36-05:00"
description = ""
images = []
menu = ""
tags = ["bdd", "chai", "javascript", "javascript", "jsdom", "mocha", "node", "npm", "npm", "nvm", "sinon", "tdd", "testing", "transmission"]
title = "Testing Transmission's Web Client, Part 1: Getting Startted with Node, Mocha, and Chai"
+++

I've been on the [Transmission](https://transmissionbt.com) project for a long time
and wrote a lot of its [web client](https://transmissionbt.com/images/screenshots/Clutch-Large.jpg).
The code's core is good,
but the rest needs some upkeep:
its interface is showing its age,
its CSS is brittle,
and it's using some unmaintained libraries.

But worst of all is there's nothing to tell us when code breaks.
For example, the "download complete" notifications are broken because they use `window.webkitNotifications` which --
in addition to being webkit-specific --
was superceded *a couple of years* ago
by the Notifications [interface](https://developer.mozilla.org/en/docs/Web/API/notification)
and [API](https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API).
Why didn't alarm bells go off when this broke?

As you've guessed from the title, the answer is tests -- or the lack of them.
This code was written before I got religion on
Test-Driven Development ([TDD](https://en.wikipedia.org/wiki/Test-driven_development))
and
Behavior-Driven Development ([BDD](https://en.wikipedia.org/wiki/Behavior-driven_development)),
so there aren't any tests.
When alarm bells dont't exist, they don't ring.

![](/images/tests-cant-fail-if-no-tests.jpg)

If we want to *know* our fixes stick,
we need to add tests as we go.

# Creating the Universe

Carl Sagan said ``If you wish to make an apple pie from scratch, you must first create the Universe.''
Since this is the code's first test, we must first create the test environment.

I'm going to use
[Mocha](https://mochajs.org/),
[Chai](https://chaijs.com/), and
[Sinon](https://sinonjs.org) for these tests.
Mocha is a JavaScript testing framework,
Chai is a BDD/TDD syntax that can be used in Mocha, and
Sinon is a tool for adding mocks and spies to tests.
These can all run together both in browsers and in Node;
for now,
I'll start with Node.

## Setting up Node

Since Transmission's web client manipulates the DOM,
we're also going to need
[jsdom-global](https://github.com/rstacruz/jsdom-global)
to set up a browserlike DOM environment to run in.
Unfortunately, the latest versions of jsdom require Node 6 or higher,
while Ubuntu 17.04 is still shipping with node 4.7.x:

    charles@calliope:~$ dpkg -s nodejs | grep "^Version"
    Version: 4.7.2~dfsg-1ubuntu3
    charles@calliope:~$ npm --version
    4.6.1

# NVM: Node Version Manager

I'd like to keep the debian-packaged version of Node on my system as-is for other projects,
but still need a newer version of Node here for jsdom.
[`nvm`](https://github.com/creationix/nvm/blob/master/README.md), the Node Version Manager, is a tool for wrangling Node installations.
After doing the [manual install](https://github.com/creationix/nvm/blob/master/README.md#manual-install)
(the automated install requires piping a magic URL into bash. No thanks!)
I get this result:

    charles@calliope:~$ nvm install node
    Downloading and installing node v8.0.0...
    Downloading https://nodejs.org/dist/v8.0.0/node-v8.0.0-linux-x64.tar.xz...
    ######################################################################## 100.0%
    Computing checksum with sha256sum
    Checksums matched!
    Now using node v8.0.0 (npm v5.0.0)
    Creating default alias: default -> node (-> v8.0.0)
    charles@calliope:~$ npm --version
    5.0.0

The shell's picking up npm 5 instead of 4.6.1.
Good!
Now we can run tests that use jsdom.

## Setting up our Node packages

The next step is to create a `package.json` file in transmission/web/
so that npm will know what packages we need and how to run tests.
First I'll create a skeleton file:

    {
      "name": "transmission-web-client",
      "description": "transmission's web client",
      "repository": "git://github.com/transmission/transmission.git",
      "version": "2.9.2",
      "license": "GPL-3.0",
      "scripts": { "test": "mocha --recursive tests/" }
    }

And then fill in the devDependencies from the command line.
As mentioned above, we want Mocha, Chai, Sinon, and jsdom.
We'll also get the mocha-jsdom and sinon-chai glue packages:

    charles@calliope:~/src/transmission/web$ npm i mocha mocha-jsdom jsdom jsdom-global chai sinon sinon-chai --save-dev

Which adds this section to `package.json`:

      "devDependencies": {
        "chai": "^4.0.1",
        "jsdom": "^11.0.0",
        "jsdom-global": "^3.0.2",
        "mocha": "^3.4.2",
        "mocha-jsdom": "^1.1.0",
        "sinon": "^2.3.2",
        "sinon-chai": "^2.10.0"
      },

Now we can run 'npm test' to confirm we've got the framework.

    # create the tests dir mentioned in package.json's 'scripts' section
    charles@calliope:~/src/transmission/web$ mkdir tests
    charles@calliope:~/src/transmission/web$ npm test

    > transmission-web-client@2.9.2 test /home/charles/src/transmission/web
    > mocha --recursive tests/

    No test files found
    npm ERR! Test failed.  See above for more details.

There aren't any tests yet,
but even a no-tests-found error is progress:
it means npm and Mocha are working.
Great!
Now let's write some JavaScript.

## Setting up our tests' scaffolding

Our first test will be a "Hello, World!" test that checks the test environment.
The goal is early detection of scaffolding problems *before* we reach the domain tests.
For example, since Transmission uses jQuery, we'll error out if `global.$` isn't set.
Let's take a look:

    var chai = require('chai');
    var expect = chai.expect;
    var sinon = require('sinon');
    var jsdom = require('jsdom-global');

    describe("Formatter", function(){
        var sandbox;
        var formatter;
        var jsdom_cleanup;

        before(function(){
            global.Transmission = {}
            formatter = require('../javascript/formatter.js')
        });

        beforeEach(function(){
            // start each test with a fresh dom
            jsdom_cleanup = jsdom();
            global.$ = require('../javascript/jquery/jquery.min.js')

            // create a sandbox with subbed console methods
            sandbox = sinon.sandbox.create();
            sandbox.stub(window.console, "log");
            sandbox.stub(window.console, "error");
        });

        afterEach(function() {
            // restore the environment
            sandbox.restore();
            // dom cleanup
            jsdom_cleanup();
        });

        it("should load", function() {
            expect(true).to.be.true;
            expect($).to.not.be.null;
        });
    });

That's a lot of code all at once.
Let's walk through it piece by piece:

    var chai = require('chai');
    var expect = chai.expect;
    var sinon = require('sinon');
    var jsdom = require('jsdom-global');

Load Chai, Sinon, and jsdom-global.
Make `expect` a local so that we don't have to always prefix it with "chai."

    describe("Formatter", function(){
        var sandbox;
        var formatter;
        var jsdom_cleanup;

        before(function(){
            global.Transmission = {}
            formatter = require('../javascript/formatter.js')
        });

Load the Formatter module just before running the first Formatter test.
Also, create an empty Transmission object because formatter requires it.
This is a wart -- ideally we wouldn't need to refer to app-domain globals -- and we should consider refactoring this away in Formatter.
Another shortcut we'll need to clean up: the hardcoded paths to formatter.js and jquery.min.js.

        beforeEach(function(){
            // start each test with a fresh dom
            jsdom_cleanup = jsdom();
            global.$ = require('../javascript/jquery/jquery.min.js')

            // create a sandbox with subbed console methods
            sandbox = sinon.sandbox.create();
            sandbox.stub(window.console, "log");
            sandbox.stub(window.console, "error");
        });

Before each test:

 * initialize the jsdom by invoking it. NB: this returns a cleanup method that we'll call in afterEach().
 * create a Sinon sandbox so that we can watch the console for log/error messages

        afterEach(function() {
            // restore the environment
            sandbox.restore();
            // dom cleanup
            jsdom_cleanup();
        });

After each test, clean up the dom and the Sinon sandbox that were created in beforeEach().

        it("should load", function() {
            expect(true).to.be.true;
            expect($).to.not.be.null;
        });

Yes! A test at last!
It's not a big test, but it's the next step on our path --
if `npm test` passes now, it means we have:

  1. npm running
  2. npm found and invoked Mocha
  3. Mocha found and loaded Chai, Sinon, and jsdom
  4. Mocha found and loaded e.g. formatter.js
  5. Our test loaded part of Formatter's outside world, e.g. jQuery's '$' global variable
  6. Our test faked part of Formatter's outside world, e.g. the Transmission global variable

What happens if we run it?

    charles@calliope:~/src/transmission/web$ npm test

    > transmission-web-client@2.9.2 test /home/charles/src/transmission/web
    > mocha --recursive tests/

      Formatter

      1 passing (499ms)

Excellent! We have a working test environment!

The next step will be to actually write Transmission-specific tests.
I'll do that in the next post in the series,
and also look at some limitations of faking a browser environment and what alternatives exist.

Other articles on Mocha and Chai that I found useful:
[1](https://nicolas.perriault.net/code/2013/testing-frontend-javascript-code-using-mocha-chai-and-sinon/)
[2](https://www.sitepoint.com/unit-test-javascript-mocha-chai/)
[3](http://chaijs.com/api/bdd/)
