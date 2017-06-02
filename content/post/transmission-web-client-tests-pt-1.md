+++
banner = "banners/backlit-keyboard.jpg"
categories = []
date = "2017-05-31T19:21:36-05:00"
description = ""
images = []
menu = ""
tags = ["bdd", "chai", "javascript", "javascript", "jsdom", "mocha", "node", "npm", "npm", "nvm", "sinon", "tdd", "testing", "transmission"]
title = "Testing Transmission's Web Client, Part One: Getting Started"
+++

I've been on the [Transmission](https://transmissionbt.com) project for a long time and wrote most of its [web client](https://transmissionbt.com/images/screenshots/Clutch-Large.jpg).
The code's core is OK,
but the rest needs some love:
the interface is showing its age,
the CSS is brittle,
and it's using some unmaintained libraries.

But the worst thing is there's nothing to tell us when the code breaks.
For example, the "download complete" notifications are broken because it uses `window.webkitNotifications` which --
in addition to being webkit-specific --
was superceded a couple of *years* ago
by the [Notifications interface](https://developer.mozilla.org/en/docs/Web/API/notification)
of the [Notifications API](https://developer.mozilla.org/en-US/docs/Web/API/Notifications_API).
Why didn't some alarm bells go off when this broke?

As you've guessed from the title, the answer is tests. Or rather, the lack of them.

This code was written before I got religion on
[TDD](https://en.wikipedia.org/wiki/Test-driven_development)/
[BDD](https://en.wikipedia.org/wiki/Behavior-driven_development),
so there aren't any tests.
When no alarm bells exist, no alarm bells ring.

If we want to *know* our code changes stick,
we need to add tests as we go.
Since we're starting from scratch,
we need to set up a test environment first.

# [If you want to make a bugfix from scratch...](https://fee.org/articles/listen-to-i-pencil-on-freakonomics-radio/)

I'm going to write these tests for [mocha](https://mochajs.org/) and [sinon](https://sinonjs.org).
Mocha is a JavaScript testing framework, and
Chai is a BDD/TDD syntax that can be used in Mocha.
Both can be used in browsers and in Node.
For now, I'll start with Node.

## Setting up Node

Since Transmission's web client manipulates the DOM,
we're also going to need
[jsdom-global](https://github.com/rstacruz/jsdom-global)
to set up a browserlike DOM environment to run in.
Unfortunately, the latest versions of jsdom require Node 6 or higher,
while my Ubuntu installation (17.04) still ships with node 4.7.x:

    charles@calliope:~$ dpkg -s nodejs | grep "^Version"
    Version: 4.7.2~dfsg-1ubuntu3
    charles@calliope:~$ npm --version
    4.6.1

# nvm

I'd like to leave the Ubuntu-installed version of npm alone,
but still use node here 6 in order to pick up jsdom.
[`nvm`](https://github.com/creationix/nvm/blob/master/README.md), the Node Version Manager, is a tool for wrangling multiple installs of node.
After doing the [manual install](https://github.com/creationix/nvm/blob/master/README.md#manual-install) -- I'd rather not pipe magic URLs into bash, thank you -- and
then running "nvm install node", I get this:

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

So, the shell's picking up npm 5 instead of 4.6.1. Now we can run tests that use jsdom.

## Setting up our Node environment

The next step is to create a `package.json` file in transmission/web/
so that Node will know what packages we need from npm and how to run tests.
First I'll create a skeleton file:

    {
      "name": "transmission-web-client",
      "description": "transmission's bundled web client",
      "repository": "git://github.com/transmission/transmission.git",
      "version": "2.9.2",
      "license": "GPL-3.0",
      "scripts": { "test": "mocha --recursive tests/" }
    }

And then fill in the devDependencies from the command line.
We want mocha, chai, sinon, and jsdom for the reasons already listed above.
We'll also add the mocha-jsdom and sinon-chai glue packages:

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

Now we can run 'npm test' to confirm we've got the framework set up.
There aren't any tests to run yet,
but just getting a no-tests-found error is progress:
it means we've got npm finding Mocha and that Mocha is looking for tests.

    # create the tests dir mentioned in package.json's 'scripts' section
    charles@calliope:~/src/transmission/web$ mkdir tests
    charles@calliope:~/src/transmission/web$ npm test

    > transmission-web-client@2.9.2 test /home/charles/src/transmission/web
    > mocha --recursive tests/

    No test files found
    npm ERR! Test failed.  See above for more details.

Great! Now let's write some JavaScript.

## Setting up our tests' scaffolding

Our first test will set up the test scaffolding, then run a simple
"Hello, World!" test that always passes. The goal here is to ensure
that the environment is getting set up properly -- for example, since
Transmission uses jQuery, we'll error out if `global.$` isn't set.
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

That's a lot of scaffolding. Let's take it piece by piece:

    var chai = require('chai');
    var expect = chai.expect;
    var sinon = require('sinon');
    var jsdom = require('jsdom-global');

Load chai, sinon, and jsdom-global.
Make an `expect` local so that we don't have to always use the chai. prefix.

    describe("Formatter", function(){
        var sandbox;
        var formatter;
        var jsdom_cleanup;

        before(function(){
            global.Transmission = {}
            formatter = require('../javascript/formatter.js')
        });

Initialize the Formatter test by loading the formatter module.
But before doing that, create an empty Transmission object because formatter requires it.
This is a wart -- ideally we wouldn't need to refer to app-domain globals -- and we should consider refactoring this away in Formatter.
We've also hardcoded an awkward path for formatter.js (and for jquery.min.js below). That needs cleanup too.

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

 * initialize the jsdom by invoking it. NB: it returns a cleanup method that we'll can call in afterEach().
 * create a sinon sandbox so that we can watch the console for log/error messages

        afterEach(function() {
            // restore the environment
            sandbox.restore();
            // dom cleanup
            jsdom_cleanup();
        });

After each test, clean up the jsdom and sinon sandboxes that we set up in beforeEach().

        it("should load", function() {
            expect(true).to.be.true;
            expect($).to.not.be.null;
        });

Yes! A test at last!
It's not a big test, but it's the next step on the path --
if `npm test` passes now, it means we have:

  1. npm finding mocha and invoking it
  2. mocha loading the chai syntax, the sinon mock tools, and jsdom
  3. mocha is finding our local assets, e.g. formatter.js, and successfully loading them
  4. we've succeeded in faking out the world that Formatter needs; e.g. the Transmission object
  5. our "should load" test itself passes, proving that true is true, and that something's in jQuery's '$' variable

What happens if we run it?

    charles@calliope:~/src/transmission/web$ npm test

    > transmission-web-client@2.9.2 test /home/charles/src/transmission/web
    > mocha --recursive tests/

      Formatter

      1 passing (499ms)

Excellent -- we have a working test environment!

In the next post in this series, I'll write some Transmission-domain tests.
We'll look at some of the limitations of running tests in Node,
and alternatives to work around those limitations.

Other articles on Mocha and Chai that I found useful:
[1](https://nicolas.perriault.net/code/2013/testing-frontend-javascript-code-using-mocha-chai-and-sinon/)
[2](https://www.sitepoint.com/unit-test-javascript-mocha-chai/)


