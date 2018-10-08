---
title: "Unit Testing AMD Modules with Karma, Mocha and RequireJS"
date: 2017-02-04T21:53:48-04:00
draft: false
type: "post"
---

As the complexity of a web application increases, manual testing of front-end functionality can no longer provide a reasonable level of confidence. Time that used to be spent developing new features is now spent fixing an increasing number of regression issues…

![](https://cdn-images-1.medium.com/max/2000/1*9jbEPTyF9zxAtOnjWCOF9Q.jpeg)

My company, like many others, is striving for daily deployments. This means that newly developed features will have less than a day to be tested before being pushed to production. This also means that we can no longer rely solely on manual testing to identify new bugs and regressions. In order to accommodate this change we’ve turned to front-end integration and unit testing.

**Integration testing** is the process of testing software modules as a group. This could mean using a tool like Selenium to comb through your website and ensure that all user functionality is working as intended (user experience tests).

**Unit testing** is the process of testing the logic in individual software modules. This can be useful when you want to make sure your validation function covers all edge cases, or even if you want feedback during the development process (TDD anyone?).

![](https://cdn-images-1.medium.com/max/2000/1*UQkdoxcM19eZmLTAaQ0N4A.jpeg)

This article will focus on the implementation of front-end unit tests. Our ‘stack’ will consist of 1) **Karma** as the test runner and 2) **Mocha** as the test framework. We’ll also make sure it plays nicely with 3) **RequireJS.**

<br/>
**Karma**

Karma is a JavaScript test runner created by the team at AngularJS. It IS possible to write unit tests with Mocha alone, however, Karma provides some powerful features that are quite nice to have. In fact, it was the lack of online resources for coupling Karma, Mocha and RequireJS that prompted me to write this article. We’ll be using Karma to build dependencies, create a test server, check for code coverage and drive tests in multiple browsers.

First, install frameworks:
<br/>
<br/>

    npm install karma --save-dev
    npm install karma-mocha --save-dev
    npm install karma-chai --save-dev
    npm install karma-requirejs --save-dev

<br/>
Karma comes with built-in plugins for Mocha, Chai and RequireJS. We’ll use Mocha for writing tests, Chai for assertions, and RequireJS for loading AMD modules. We’ll look at these in more detail later.

Next, install plugins (and PhantomJS):
<br/>
<br/>

    npm install karma-chrome-launcher --save-dev
    npm install karma-firefox-launcher --save-dev
    npm install karma-opera-launcher --save-dev
    npm install karma-safari-launcher --save-dev
    npm install karma-phantomjs-launcher --save-dev
    npm install phantomjs --save-dev

<br/>
Karma is powerful in that it allows tests to be run simultaneously in multiple browsers. We’ll be supporting Firefox, Opera, Chrome, Safari and PhantomJS. These browser plugins are needed to help Karma find the appropriate binary files, which means you can only run tests on browsers that are already installed on your local machine.

Oh, and while you’re at it, throw in karma-coverage
<br/>
<br/>

    npm install karma-coverage --save-dev

<br/>
This plugin is a fork of [Istanbul](http://gotwarlost.github.io/istanbul/) which will give us the ability to easily track code coverage.

<br/>
**Karma**

Next, create your karma config file. This can be done with the command “karma init” or by creating a karma.conf.js file in your apps root directory. This is what my config file looks like:
<br/>
<br/>

    module.exports = function(config) {
      config.set({

        basePath: '',

        frameworks: ['mocha', 'requirejs', 'chai'],

        files: [
            {pattern: './config.js', included: true},
            {pattern: './app/**/*.json', included: false},
            {pattern: './app/**/*.html', included: false},
            {pattern: './app/**/*.js', included: false},
            {pattern: './library/**/*.js', included: false},
            {pattern: './unit-tests/**/*.js', included: false},
            {pattern: './unit-tests/main.js', included: true}
        ],

        exclude: [
            './app/private/header/failure.js'
        ],

        preprocessors: {
            './app/**/*.js': 'coverage'
        },

        coverageReporter: {
          type : 'text-summary',
          dir : 'coverage/',
          includeAllSources : true
        },

        reporters: ['progress', 'junit', 'coverage'],

        browsers: ['PhantomJS', 'Chrome', 'Firefox', 'Safari'],

        port: 9876,

        colors: true,

        logLevel: config.LOG_INFO,

        autoWatch: true,

        singleRun: false,

        concurrency: Infinity,
      })
    }

<br/>
It is important to understand what is happening in the Karma config file:
<br/>
<br/>

    basePath: '',
    frameworks: ['mocha', 'requirejs', 'chai'],

<br/>
***basePath*** is used to resolve all relative paths defined in the *files* and *excludes* properties of the config.
<br/>
<br/>

    files: [
        {pattern: 'config.js', included: true},
        {pattern: 'app/**/*.json', included: false},
        {pattern: 'app/**/*.html', included: false},
        {pattern: 'app/**/*.js', included: false},
        {pattern: 'libs/**/*.js', included: false},
        {pattern: 'unit-tests/**/*.js', included: false},
        {pattern: 'unit-tests/main.js', included: true}
    ],

<br/>
***files*** tells Karma which files to serve when it starts it’s web server. Any files that need to be imported, tested, or seen in any capacity need to included here. File patterns leverage the [minimatch](https://github.com/isaacs/minimatch) library — i.e. ```app/**/*.js``` will include all files in all subdirectories of the app directory. Only set *included: true* for files that should be included as a ```<script>``` tag or you may run into errors with RequireJS.
<br/>
<br/>

    exclude: [
        './app/failure.js'
    ],

<br/>
***exclude*** lets you specify files that you don’t want Karma to serve. This is useful when you want to use minimatch to include an entire directory minus a few exceptional files.
<br/>
<br/>

    preprocessors: {
        './app/**/*.js': 'coverage'
    },

    coverageReporter: {
        type : 'text-summary',
        dir : 'coverage/',
        includeAllSources : true
    },

    reporters: ['progress', 'junit', 'coverage'],

<br/>
This chunk of the config is responsible for reporting and code coverage. The code to be checked for coverage is specified in ***preprocessors*** and coverage options can be modified in the ***coverageReporter.*** The desired output format is then specified in ***reporters*** (in this case, ‘progress’ displays the number of tests executed, ‘junit’ pipes the results to an xml file and ‘coverage’ logs a summary of coverage results.
<br/>
<br/>

    browsers: ['PhantomJS', 'Chrome', 'Firefox', 'Safari', 'Opera'],

    port: 9876,

    colors: true,

    logLevel: config.LOG_INFO,

    autoWatch: true,

    singleRun: true,

    concurrency: Infinity,

<br/>
In this example, the ***browsers*** we are testing include PhantomJS, Chrome, Firefox, Safari and Opera. Karma will run a web server on ***port*** 9876. Output log ***colors*** are enabled with a ***logLevel*** of “info”. The server is watching (***autoWatch: true***) for file changes automatically. Each run will test all browsers simultaneously (***concurrency: Infinity***) and finish by exiting all browsers/karma (***singleRun: true***).

<br/>
**RequireJS**

Most of the work to set up RequireJS has already been done if you’ve followed up to this point. We’ve already included the RequireJS config file (see snippet below) and Karma has now been configured to bring in RequireJS as a framework.
<br/>
<br/>

    files: [
        {pattern: './config.js', included: true},
        {pattern: 'unit-tests/main.js', included: true}
    ]

<br/>
There is some final configuration that needs to be done, which we’ll do in the main.js file.
<br/>
<br/>

    var tests = [];

    for (var file in window.__karma__.files) {
        if (/\_test\.js$/.test(file)) { tests.push(file); }
    }

    requirejs.config({
        baseUrl: '/base/app/',
        deps: tests,
        callback: mocha.run
    });

<br/>
We’re leveraging a bit of regex and the fact that karma serves all of its files in ```window.__karma__.files``` to specify all of our test files (which we label with _test). Calling mocha.run after the ```requirejs.config()``` update will initialize testing!

If you’ve followed along with everything up until now, you’re ready to start writing unit tests — just label them with “_test.js”. Once you have written some tests, running them is as simple as typing ```karma start karma.conf.js``` in the command line (you might need to specify a path if you haven’t installed karma globally — ```/node_modules/karma/bin/karma start karma.conf.js```).

Happy testing!

**Update (11/18/2017):**

Per requested, here is what my directory tree looks after setup:

![Example unit test directory with karma.conf.js / main.js / *_test.js failes](https://cdn-images-1.medium.com/max/2000/1*NahHIVQ268tO9IJp1Vuo0A.png)<center>*Example unit test directory with ```karma.conf.js / main.js / *_test.js```*</center>

This particular app is built with Backbone.js so I have folders split by role (i.e. admin) and collections/models/views. It doesn’t matter where you put main.js as long as you specify the correct path in the Karma configuration file.

Here’s what the actual test looks like:

![Unit testing a simple Backbone.js view — instantiation of the view in before(), checking that it is a subclass of the base view and testing isValidIPv4().](https://cdn-images-1.medium.com/max/3232/1*RoPWVQhlGTKokYMAaven9A.png)<center>*Unit testing a simple Backbone.js view — instantiation of the view in ```before()```, checking that it is a subclass of the base view and testing ```isValidIPv4()```.*</center>

This is a simplistic test of a backbone view. We use the mocha ```before()``` hook to instantiate the view (AddIpDlgView.js), check if it’s a subclass of the base view and test the validity of a few IPs with ```addIpDlgView.isValidIPv4()```. Dependencies are imported using typical ```define()``` syntax.

Comment on [Medium](https://medium.com/@BoweiHan/setup-for-testing-amd-modules-with-karma-requirejs-and-mocha-2be3931c6a72)