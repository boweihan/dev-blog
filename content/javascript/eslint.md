---
title: "Instantly become a better developer with ESLint"
date: 2017-11-05T21:53:48-04:00
draft: false
type: "post"
---

I spent a few months writing JavaScript code without a linter. Back then, it felt normal to deal with run-time errors by checking the browser console — messages containing…
<br/>
<br/>

```
Uncaught SyntaxError: Unexpected token
```

<br/>
…were a normal part of my development process. My code was full of poor practices (even when it worked!) — declaring functions inside of loops, unused variables, poor spacing, dangerous coercion — and the list goes on.

![Real life lint errors](https://cdn-images-1.medium.com/max/2000/0*s5ox5PugBL8KVZQh.png)<center>*Real life lint errors*</center>

The day I installed my first linter (JSHint) was the day I finally walked out of the cave. In a single instant, my code quality increased markedly.
<br/>
<br/>

* I could now catch most of my run-time errors in my editor

* My code was more meaningful and less brittle

* I wasn’t falling into common JS traps

<br/>
And all that…for free! My linter was teaching me things that I never knew I needed to know and correcting me as I smashed my face against the keyboard.

![](https://cdn-images-1.medium.com/max/2000/0*QP4Be4aFdqOLTCB1.gif)

To this day I still feel that installing a linter is like getting free coding “skill points” — there’s NO reason not to be linting your JavaScript code! Here’s how.

ESLint is the linter I’ve chosen to go with for this post. I find that it’s relatively user-friendly and best of all — configurable. It’s great for a variety of projects and comes with a bunch of templates and plugins out of the box!

First things first — install ESLint (I assume you have a project with NPM initialized)
<br/>
<br/>

```
npm install --save-dev eslint
```

<br/>
We’re installing ESLint as a dev dependency. Once ESLint is installed, you can set up a configuration file:
<br/>
<br/>

```
eslint --init
```

<br/>
You’ll get the following choices:

![](https://cdn-images-1.medium.com/max/2296/0*fyP7DZ0jTCJzy0yz.png)

If you’re comfortable answering questions about your codebase, feel free to select option (1). It’ll ask you the following questions:
<br/>
<br/>

* Are you using ES6 features?

* Where will your code run?

* Do you use CommonJS

* What style of indentation do you use?

* What quotes do you use for strings?

* What line endings do you use?

* Do you require semicolons?

* What format do you want your config file to be in?

<br/>
My goal from this post is to get your linter up and running as quickly as possible, so lets choose option (2) — “**Use a popular style guide**”.

This will pop up a new menu:

![](https://cdn-images-1.medium.com/max/2428/0*qKlPR6bIGR8NRa6q.png)

I personally think it’s awesome that the Google, Airbnb and StandardJS linting rulesets are provided by ESLint out of the box. They are all amazing rulesets for first-time users so feel free to choose any of the three.

**Airbnb — [https://github.com/airbnb/javascript](https://github.com/airbnb/javascript)**

Airbnb is a comprehensive ruleset that brands itself as a “mostly reasonable approach to JavaScript”. Airbnb provides a bunch of ESLint presets that cover ES6, JSX, (etc) that makes it a great choice for React projects. It also helps by providing performance tips!

**Google — [https://google.github.io/styleguide/jsguide.html](https://google.github.io/styleguide/jsguide.html)**

Google’s style-guide is widely shared and focuses on two sections — (1) JavaScript language rules (variables, functions) and (2) JavaScript style rules (cosmetic).

**StandardJS — [https://standardjs.com/](https://standardjs.com/)**

Branded as “one style to rule them all”, StandardJS is (unsurprisingly) trying to become the standard linting ruleset. I haven’t worked with it much but it seems like a minimalistic ruleset that has been gaining some momentum.

![](https://cdn-images-1.medium.com/max/2400/0*4XMqub13eTov11ml.png)

I’m going to select Airbnb since we’re working with React and ES6. Once you select that option make sure to select YES to React and choose JavaScript for the config format.

![](https://cdn-images-1.medium.com/max/3592/0*Uhe69ian3mN61BM_.png)

The necessary plugins for the Airbnb guide will be installed automatically.

Congrats! You’ve installed ESLint. But wait…how do I use this thing??

<br/>
**Linting through the command line**

Since we’ve only installed ESLint locally, we’ll pull the command line tool from node_modules (you won’t just be able to type “eslint” in your command line unless you have the appropriate binaries and packages installed globally). The path to the ESLint command line tool is:
<br/>
<br/>

```
./node_modules/eslint/bin/eslint.js
```

<br/>
You can lint files one at a time with this tool:

![Yeah…I probably should have written better JavaScript](https://cdn-images-1.medium.com/max/3836/0*kD_qdbVJiC6l7k8E.png)<center>*Yeah…I probably should have written better JavaScript*</center>

Simply pass your filenames as an argument to eslint.js. Alternatively, you can also use file matching patterns that will lint a larger subset of your files.

![Oops…probably shouldn’t lint node_modules](https://cdn-images-1.medium.com/max/3920/0*C3GtkSEW9wisZc_h.png)<center>*Oops…probably shouldn’t lint node_modules*</center>

Linting through the command line is great when you want to do things like build automation, but personally I prefer to catch lint errors as I’m writing my code. The best way to do this is to install an editor plugin!

<br/>
**Linting from your favorite editor**

Almost every popular editor has a plugin available for ESLint in this day and age. I personally use **Atom** (haters gonna hate) for most of my JS development and the plugin that I use is called linter-eslint.

![](https://cdn-images-1.medium.com/max/2940/0*Cy5oni51TIT1Ou9K.png)

Simply install that sucker and it’ll do all the heavy lifting for you. Typical editor plugins for ESLint will find your config file (.eslintrc) and show — realtime — exactly where you are writing lint errors into your code. It will look a little like this (you might need to restart your editor for the plugin to work):

![Okay….that’s a lot of errors.](https://cdn-images-1.medium.com/max/3416/0*0jEQW8pv7GUUdnuK.png)<center>*Okay….that’s a lot of errors.*</center>

Which, coincidentally, is a copy of my ESLint file being linted by another ESLint configuration (INCEPTION…not really).

<br/>
**Congratulations, you’ve just upped your coding game!**

If this is the first time you’ve installed a linter — congrats! Your code is now better and you should be happier as you write it. If you have any questions, or if this guide did NOT get you all the way to a working linter please let me know and I’ll do my best to help you out!

Comment on [Medium](https://medium.com/@BoweiHan/instantly-improve-your-code-by-setting-up-eslint-in-your-react-npm-project-34face702cbe)