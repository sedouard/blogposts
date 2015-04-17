# Running Ember Test Automation in Node Webkit

Recently last month my buddy **[Felix](http://www.felixrieseberg.com)** and I decided to start working on a project to bring functionality of the windows-only **[azure storage explorer](https://azurestorageexplorer.codeplex.com)** to a **[cross platform implementation](http://github.com/azure-storage/xplat)** that will work on any operating system.

We decided to use **[Ember](http://emberjs.com)** MVC framework and specifically the **[Ember CLI](http://ember-cli.com)** build tooling with node-webkit. It turns out that there was a a really handy **[Ember-CLI plugin](https://github.com/brzpegasus/ember-cli-node-webkit)** made by **[Estelle DeBlois](https://github.com/brzpegasus)** over at **[Dockyard.com](http://dockyard.com)** to run ember-cli applications within the node-webkit environment.

## Ok... What's Node-webkit? 

If you're unfamiliar with projects like **[node-webkit](https://github.com/nwjs/nw.js/)**, now known as simply **nwjs** or **[atom shell](https://github.com/atom/atom-shell)** these are tools which allow you to run node.js code within the context of a browser. So think of something like this which would be totally impossible in a normal browser, suddenly becomes possible:

<script src="https://gist.github.com/sedouard/98ced31654e91a7f7360.js"></script>

All the sudden, you have everything you need to make a true native application since you have access to OS-level components like the file system and processes with the UI implemented with web technologies you already know and love.

Combined with the Ember MVC framework we have a full cross-platform native application development stack.

## Running the Tests

As every good developer knows, having tests keep your project sane. The problem that we ran into as we developed our project is that there was no way to run our test automation within the Node-webkit environment. Out of the box, `ember test` is designed to run in **[phantomjs](http://phantomjs.org)** which has no idea what the hell node.js is.

In order to run any kind of integration test, we would need to run our tests in `nwjs` to ensure proper integration between our code in node.js and in the browser.

## How You Can Run Your Ember Tests in Nw.js

I recently partnered with Estelle to get a **[pull request](https://github.com/brzpegasus/ember-cli-node-webkit/pull/13)** into her **[awesome ember-cli plugin](https://github.com/brzpegasus/ember-cli-node-webkit)** which solves this problem for anyone who adds this plugin to their application.

Now, if you want to write an emberjs application for node-webkit here's what you do:

<script src="https://gist.github.com/sedouard/8a12a6a1d4e442c8c86e.js"></script>

As you write your unit tests, you can test ember adapters, controllers and do integration tests all within nw.js allowing you to run your real-world node.js code within your test environment.

Although its a good idea to keep your node.js code and your browser code very seperate it's super valuable to be able to run tests in the same environment as your customer.

## Running Ember Nw.js CLI Tests in Travis-CI

You can run your application's test automation in build systems like **[Travis CI](http://travis-ci.org)** by specifying to the travis build environment that an **[Xframe Virtual Buffer](http://en.wikipedia.org/wiki/Xvfb)** or `xfvb` is required.

To do this in your `.travis.yml` add this line:

<script src="https://gist.github.com/sedouard/257247443cbad3ea086c.js"></script>

This will instruct travis to startup a virtual buffer which simulates a native window for an application. In our case `nwjs` will think its running in a window, (when in actuality, it is not)**. By doing this we can get our `nwjs` tests running in continuous integration!

Here's a log of what **[our Travis-CI build](https://travis-ci.org/azure-storage/xplat)** looks like. Ignore the scary warnings! They are beningn:

<script src="https://gist.github.com/sedouard/7599379d72477f3f8c3c.js"></script>

Now you can be more confident that your ember nwjs application works upon checkin.

Happy Hacking!