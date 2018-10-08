---
title: "JavaScript Closures Demystified"
date: 2017-11-30T21:53:48-04:00
draft: false
type: "post"
---

Kyle Simpson, writer of the series “You Don’t Know JS”, said the following — “understanding closures is like when Neo sees the Matrix for the first time”. If you don’t understand the reference please come back after you’ve watched The Matrix (1999). I admit this concept took me a while to grasp and for a long time I walked around with inaccurate beliefs about what a closure really was!

![](https://cdn-images-1.medium.com/max/2000/0*M5VWy0aC-n1fuc-S.)

Wikipedia has the following definition for closure:

<i>In programming languages, **closures** (also **lexical closures** or **function closures**) are techniques for implementing lexically scoped name binding in languages with first-class functions. Operationally, a closure is a record storing a function together with an environment: a mapping associating each free variable of the function (variables that are used locally, but defined in an enclosing scope) with the value or reference to which the name was bound when the closure was created. A closure — unlike a plain function — allows the function to access those captured variables through the closure’s copies of their values or references, even when the function is invoked outside their scope.</i>

Now there’s a mouthful! Since I think this is actually a pretty great definition for a closure, let’s break it down.

***In programming languages, closures (also lexical closures or function closures) are techniques for implementing lexically scoped name binding in languages with first-class functions.***

Two bombs get dropped in this sentence — first-class functions and lexical scoping. A language has **first-class functions** if it allows functions to have behavior that is similar to other entities (i.e. an integer) — this typically means that functions can be passed as an argument, returned from a function, modified, and assigned. Have you ever played with JavaScript code that looks like this?
<br/>
<br/>

    _.partial = function(func) {
        var boundArgs = slice.call(arguments, 1);
        var bound = function() {
          var position = 0, length = boundArgs.length;
          var args = Array(length);
          for (var i = 0; i < length; i++) {
            args[i] = boundArgs[i] === _ ? arguments[position++] :    boundArgs[i];
          }
          while (position < arguments.length) args.push(arguments[position++]);
          return executeBound(func, bound, this, this, args);
        };
        return bound;
      };

<br/>
This example might be a bit complex, but the reason I chose it is because it’s the source of a popular utility function _.partial from a very popular JavaScript library **Underscore.js**. This is a great example of what you can do with **first-class functions** — a function is being passed as an argument to a function which is also returning a function!

The term “lexical” in **lexical scope** has an English definition (*relating to the words or vocabulary of the language*) but it makes more sense to think in terms of a compiler. The first step of a standard language compiler is called **lexing** in which source code is parsed and assigned a semantic meaning. **Lexical scope** is scope that is defined by the JavaScript compiler at lexing time. In simpler terms, lexical scope refers to the scope that you (the programmer) create as you “physically” write code. This usually manifests as blocks of code (and sometimes indentations) that can be obvious from simply reading the code.
<br/>
<br/>

    function outerScope(a) {
        var b = a / 2;
        function innerScope(c) {
            console.log(a, b, c);
        }
        innerScope(b/2);
    }
    outerScope(4); // 4, 2, 1

<br/>
Here there are three levels of **lexical scope** — the global scope (which contains outerScope()), the scope created within outerScope (which contains innerScope) and the scope created by innerScope. The key here is that **lexical scope** can be derived from reading the source code, which in this case is clearly presented by the nested functions. This is in contrast to **dynamic scope** which depend on how the code is executing (note: the *this* keyword in JavaScript has dynamic scoping determined by execution context). Lexical scope means that when we execute innerScope(), it is also able to access it’s surrounding environment (i.e. a, b and c).

![](https://cdn-images-1.medium.com/max/2000/0*Ou5iLB6K2vV_vcaj.png)

<center><i>A closure is a combination of a function and the lexical environment within which that function is called” — MDN web docs</i></center>

So now, armed with our newfound knowledge, we can summarize the first sentence of Wikipedia’s closure definition as — “*In programming languages which treat functions as a first-class citizen, closures allow functions to be invoked with a complete memory of it’s original lexical environment*”. This is pretty much summed up by Wikipedia’s second and third sentence:
> ***Operationally, a closure is a record storing a function together with an environment: a mapping associating each free variable of the function (variables that are used locally, but defined in an enclosing scope) with the value or reference to which the name was bound when the closure was created. A closure — unlike a plain function — allows the function to access those captured variables through the closure’s copies of their values or references, even when the function is invoked outside their scope.***

Here’s an example of a closure that demonstrates how variables are mapped:
<br/>
<br/>

    var counter = (function() {
        var count = 0;
        function increment() {
            count += 1;
        }
        return {
            increment: function() {
                increment(1);
            },
            getValue: function() {
                return count;
            }
        }
    })();

    console.log(counter.getValue()); // 0
    counter.increment();
    console.log(counter.getValue()); // 1
    counter.increment();
    console.log(counter.getValue()); // 2
    counter.increment();
    console.log(counter.getValue()); // 3

<br/>
Here we are calling counter.getValue() and counter.increment() which both retain a reference to the count variable, even though we’re calling these functions from the global scope which doesn’t have a reference to count! This is a great example of how we would use closures to make private variables or hide implementation details. The closure is mapped to the lexical environment that is created by the body of the anonymous function, which allows count to remain reference to counter.

Here’s another interesting example of what a closure can accomplish:
<br/>
<br/>

    for (var i = 0; i <= 5; i++) {
        setTimeout(function timer() {
            console.log(i);
        }, i * 1000);
    }

    // 6
    // 6
    // 6
    // 6
    // 6

<br/>
Psych!! There’s actually no (meaningful) closure here. it may look like setTimeout is retaining a reference to the variable i but in reality i is actually being assigned to the global scope where it is being referenced by the console.log statement. Here’s how we could actually make this work as intended (even with the setTimeout!):
<br/>
<br/>

    for (var i = 1; i <= 5; i++) {
        (function() {
            var j = i;
            setTimeout(function timer() {
                console.log(j);
            }, j * 1000);
        })();
    }

    // 1
    // 2
    // 3
    // 4
    // 5

<br/>
This time we’re actually using an anonymous function (or IIFE — immediately invoked function expression) to create a closure in which we’re assigning a variable j which is equal to i at the time of invocation. Each new variable j is then mapped to a new closure along with its own setTimeout function. The really cool thing here is that even though the IIFE function is long-gone by the time the setTimeout function runs, the variable j is still in play!

![](https://cdn-images-1.medium.com/max/2000/0*1t2amWglPugvtyLx.jpg)

Hopefully this post starts to clear up some of the mystery around closures. Closures are a major part of functional programming that help with the creation of function factories, name-spacing, complexity management, abstraction and much more!

Comment on [Medium](https://medium.com/front-end-hacking/javascript-closures-54edf12606c4)
