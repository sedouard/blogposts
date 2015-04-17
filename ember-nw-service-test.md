# Mocking Ember Services for Unit Testing

Let's be honest. Testing is not sexy. To be honest I think this is about as sexy testing is going to get. So let's just get it over with :-)

We're working slowly but surely on a **[cross-platform azure storage explorer](http://github.com/azure-storage/xplat)** in order to manipulate **[azure blobs](http://azure.microsoft.com/en-us/documentation/services/storage/)** and soon tables and queues.

The explorer is a unique ember-cli **[node-webkit](https://github.com/nwjs/nw.js/)** application in which we use the azure storage client api, which is really intended to be used on a server. However, if you take a look at the full **[windows version](http://azurestorageexplorer.codeplex.com/)**, it's used as a tool for developers of cloud services to manage their data in their storage account. 

If you aren't familiar with **[node-webkit](https://github.com/nwjs/nw.js/)** you can checkout my **[previous blog post](http://blog.stevenedouard.com/running-ember-test-automation-in-node-webkit-nwjs/). Basically it allows us to mix node.js with traditional front-end javascript, making for a cross-platform native development stack.

There's already a full featured **[node.js client api](http://npmjs.org/azure-storage)** for azure storage and it only made sense for us to use that.

## Custom Ember Adapters

**[Ember](http://emberjs.com)** out of the box comes with a REST adapter which assumes that your back-end server api follows **[restful convention](http://www.restapitutorial.com/lessons/restfulresourcenaming.html). Sometimes when your backend api doesn't conform exactly you can **[extend the ember REST adapter](http://guides.emberjs.com/v1.10.0/models/the-rest-adapter/)** to adapt the the behavior of your specific back-end.

In our case we wanted to circumvent ember's pre-baked REST api adapter behavior because in actually we didn't plan on talking to a RESTful api, rather the **[azure storage client for node.js](http://npmjs.org/azure-storage)** which in-turn would use the azure storage api.

This required us to replace the entire REST adapter with a custom adapter fitted for the node.js client library - which makes for a very interesting use case.

## Unit Testing Our Components

Our project uses **[Ember-cli](http://ember-cli.com)** a command line interface which helps build and maintain your ember app. Out-of-the-box our application comes with **[testem](https://github.com/airportyh/testem)** and ember **[QUnit]()** testing framework. 

One way we keep our Node.js side of code seperated from our front-end javascript code is through the use of **[ember services](https://github.com/rwjblue/ember-qunit)**. Ember services allows you to present a concept called a service to many components. In our case our custom adapter uses an ember service which wraps our Node.js code.

Here's what our **[blob adapter](https://github.com/azure-storage/xplat/blob/master/app/adapters/blob.js)** looks like:

<script src="https://gist.github.com/sedouard/6b9ac6c768bdb2bd7e3c.js"></script>

Notice, how we call `Ember.inject.service()` which is what provides access to our nodejs code to the field `nodeServices`.

We set `azureStorage` to the azure storage service which essentially implements the **[azure storage for node.js client api](http://npmjs.org/azure-storage)**.

Now here's what our Ember node service looks like:

<script src="https://gist.github.com/sedouard/2b827ca4525a3f00b812.js"></script>

This is super simple! `requireNode` is an alias to `require` for AMD modules in node.js. We essentially exposed the node modules `fs` and `azure-storage` through this service. It's not immediatly apparent why this is great until we get to the testing story.

## Mocking the Storage Services

Because we've implemented this as an ember service, we have the flexibilty of overriding the service implementation by mocking the service and thus isolate our testing to just the units we care about.

Here's what ember 'node service' **[implementation]()** looks like for our unit tests that we regularly run in **[travis-ci](https://github.com/azure-storage/xplat/blob/master/tests/helpers/start-app.js)** (abridged version):

<script src="https://gist.github.com/sedouard/808288c0c0d82276814e.js"></script>

We've essentially implemented a mocked version of the expected apis found in **[azure-storage](https://npmjs.org)** so now we can run our controllers, models and adapters in isolation!

Further, notice how we have `assert` on every api call. This allows us to inherntly validate with some certainty that the component(s) under test correctly call the api. It's almost like getting free testing!

Now when we call run, for example, our **[explorer controller tests](https://github.com/azure-storage/xplat/blob/master/tests/unit/controllers/explorer-test.js)**, we can validate that it has interacted with our api correctly without having to do much:

<script src="https://gist.github.com/sedouard/217c4f997b1eaab49970.js"></script>

## What Does This All Mean?

Mocking isn't a new concept in testing, and emberjs makes it really easy to implement mocking via **[ember services]()**. This doesn't just apply to **[node-webkit]()** applications, **[azure]()** applications but ANY application.

You can use ember services to implement any mocking of a remote system that you'd like to not take a dependency in your continuous integration tests.

And when we're ready to run a live test, we just remove the mocked service implementation and run the test suite live. In our case we **[can even run our test automation headless or not within node-webkit](http://blog.stevenedouard.com/running-ember-test-automation-in-node-webkit-nwjs/)**.

Happy coding!