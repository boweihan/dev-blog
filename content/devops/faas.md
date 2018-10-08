---
title: "An Introduction to Serverless and FaaS (Functions as a Service)"
date: 2017-12-15T21:53:48-04:00
draft: false
type: "post"
---

When was the last time you had your mind blown? For me, it was last Friday. I attended a local conference (shoutouts to Functions17!) that hosted some outstanding speakers from innovative companies such as Amazon, Google, Pivotal, and Microsoft…for the sole purpose of introducing and discussing innovations for a new software architecture known as FaaS, or Functions as a Service.

![](https://cdn-images-1.medium.com/max/2000/0*E262CkJzKmU1u4EB.jpg)

<br/>
**Serverless Computing**

Before we get into what FaaS is, it’s important to understand another term — **serverless computing**. Serverless computing is a cloud computing model which aims to abstract server management and low-level infrastructure decisions away from developers. In this model, allocation of resources is managed by the cloud provider instead of the application architect, which can bring some serious benefits. In other words, serverless aims to do exactly what it sounds like — allow applications to be developed without concerns for implementing, tweaking, or scaling a server (at least, to the perspective of a user).

<br/>
**Functions as a Service**

**FaaS** is a relatively new concept that was first made available in 2014 by hook.io and is now implemented in services such as AWS Lambda, Google Cloud Functions, IBM OpenWhisk and Microsoft Azure Functions. It provides a means to achieve the serverless dream allowing developers to execute code in response to events without building out or maintaining a complex infrastructure. What this means is that you can simply upload modular chunks of functionality into the cloud that are executed independently. Imagine the possibilities! Instead of scaling a monolithic REST server to handle potential load, you can now split the server into a bunch of functions which can be scaled automatically and independently. If you’re familiar with microservices, this image might make you feel something (credits to Luke Angel(1)).

![](https://cdn-images-1.medium.com/max/2000/0*Yv6sMLN_7lFjdH3I.png)

As an illustration, here’s all the code you would need to deploy a fully scalable and useable function to the cloud.
<br/>
<br/>

    /**
     * HTTP Cloud Function.
     *
     * @param {Object} req Cloud Function request context.
     * @param {Object} res Cloud Function response context.
     */
    exports.helloHttp = function helloHttp (req, res) {
      res.send(`Hello ${req.body.name || 'World'}!`);
    };

<br/>
This is a simple HTTP request written in NodeJS that displays “Hello World” or “Hello (name)” if you pass in a parameter. This example is borrowed from Google Cloud which provides some useful features such as type safety (via @param).

Here’s another snippet that is a little more involved. Can you guess what it does?
<br/>
<br/>

    /**
     * Background Cloud Function to be triggered by Cloud Storage.
     *
     * @param {object} event The Cloud Functions event.
     * @param {function} callback The callback function.
     */
    exports.helloGCS = function (event, callback) {
      const file = event.data;

      if (file.resourceState === 'not_exists') {
        console.log(`File ${file.name} deleted.`);
      } else if (file.metageneration === '1') {
        // metageneration attribute is updated on metadata changes.
        // on create value is 1
        console.log(`File ${file.name} uploaded.`);
      } else {
        console.log(`File ${file.name} metadata updated.`);
      }

      callback();
    };

<br/>
This is an example of a function that can react to changes in a Google Cloud Storage bucket (i.e. file upload, deletion and metadata changes). If you want a complete example of how to implement this hit up [https://cloud.google.com/functions/docs/tutorials/storage.](https://cloud.google.com/functions/docs/tutorials/storage.)

It’s pretty damn cool that we can simply write and deploy a JavaScript function and be confident in its availability, scalability AND cost-efficiency!

<br/>
**FaaS Advantages**

1. Fewer developer logistics — server infrastructure management is handled by someone else.

1. More time focused on writing code / app specific logic — higher developer velocity.

1. Inherently scalable. Rather than scaling your entire application you can scale your functions automatically and independently with usage.

1. Never pay for idle resources.

1. Built in availability and fault tolerance.

1. Business logic is necessarily modular and conform to minimal shippable unit sizes.

<br/>
**FaaS Disadvantages**

1. Decreased transparency. Someone else is managing your infrastructure so it can be tough to understand the entire system.

1. Potentially tough to debug. There are tools that allow remote debugging and some services (i.e. Azure) provide a mirrored local development environment but there is still a need for improved tooling.

1. Auto-scaling of function calls often means auto-scaling of cost. This can make it tough to gauge your business expenses.

1. You now have a ton of functions deployed and it can be tough to keep track of them. This comes down to a need for better tooling (**developmental: **scripts, frameworks, **diagnostic:** step-through debugging, local runtimes, cloud debugging, and **visualization:** user interfaces, analytics, monitoring).

1. Solutions for caching resources between stateless requests (i.e. DB connections) and recycling network requests are still pending.

It’s important to note that FaaS isn’t a hammer that you can use for every nail. That being said, many of its disadvantages can be attributed to its infancy.

<br/>
**FaaS use cases**

Web apps, Backends, Data/stream processing, Chatbots, Scheduled tasks, IT Automation

<br/>
**Existing FaaS providers**

* Microsoft/Azure Functions — [https://azure.microsoft.com/en-us/services/functions/](https://azure.microsoft.com/en-us/services/functions/)

* Tooling, bindings & triggers

* Logic apps — visual designer with 25+ connectors and function orchestration

* Event grid

* Local debugging

* AWS Lambda Functions — [https://aws.amazon.com/lambda/](https://aws.amazon.com/lambda/)

* Functions, APIs, Tables

* Google Cloud Functions / Firebase — [https://cloud.google.com/functions/](https://cloud.google.com/functions/)

* Cloud storage, Cloud pub/sub, HTTPS, Stackdriver logs

* Tracing & debugging in production

* Firebase provides realtime DB, analytics, additional event triggers

* IBM Cloud Functions — [https://console.bluemix.net/openwhisk/](https://console.bluemix.net/openwhisk/)

<br/>
**Libraries supporting FaaS**

**stdlib** — [https://stdlib.com/](https://stdlib.com/)

* Positions themselves as the “standard library for Functions as a Service”

* High level framework that tries to standardize FaaS providers and abstract it to the point where you only need to worry about 1) functions and 2) API specifications.

* Supports type safety with NodeJS using an extended set of types (hello float)

* Easy to ship code

**spring-cloud-function** — [http://cloud.spring.io/spring-cloud-function/](http://cloud.spring.io/spring-cloud-function/)

* Builds on Spring Boot to promote functional programming in Java

**projectreactor.io** — [http://projectreactor.io/](http://projectreactor.io/)

* Java library for building reactive applications

**@architect** — [https://github.com/arc-repos/arc-workflows](https://github.com/arc-repos/arc-workflows)

* Simple JavaScript library that provides scripts to create/deploy and modify cloud functions

<br/>
**Conclusion**

I’m quite new to this FaaS revolution but the pieces that I’ve seen are pretty damn cool. I think I need to learn a bit more to write anything insightful into this “conclusions” section but I hope this post has at least piqued your interest in something new and exciting.

Happy learning!

<br/>
***Sources/Reading***

*1 — [http://lukeangel.co/cross-platform/docker-servless-faas-functions-as-a-service/](http://lukeangel.co/cross-platform/docker-servless-faas-functions-as-a-service/)*

*2- [https://martinfowler.com/articles/serverless.html](https://martinfowler.com/articles/serverless.html)*

*3- [https://www.devbridge.com/articles/what-comes-after-microservices-serverless/](https://www.devbridge.com/articles/what-comes-after-microservices-serverless/)*

*4- [https://www.iron.io/what-is-serverless-computing/](https://www.iron.io/what-is-serverless-computing/)*

*5- [https://stackify.com/function-as-a-service-serverless-architecture/](https://stackify.com/function-as-a-service-serverless-architecture/)*

<br/>
**To Readers: What are your thoughts on Serverless and FaaS? Is this the future?**

Comment on [Medium](https://medium.com/@BoweiHan/an-introduction-to-serverless-and-faas-functions-as-a-service-fb5cec0417b2)