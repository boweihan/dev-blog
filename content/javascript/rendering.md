---
title: "Optimizing React Performance with Selective Rendering"
date: 2018-10-17T13:18:48-05:00
draft: false
type: "post"
---

<h1 style="color: darkgray">Avoiding Avoidable Component Re-renders</h1>

<br/>
My team recently spent some time making performance improvements to a data-reporting front-end we build and maintain at [Vena Solutions](https://venasolutions.com/). The application is built entirely in React and has grown to the point where we are starting to notice sluggish performance in certain views and features — especially those tasked with rendering many sub-components (lists and trees). It turns out that most of our issues were the result of sub-optimal key usage and avoidable component re-rendering.

This article highlights some of the optimizations that we applied to reduce avoidable re-renders in our application.

<br/>

## **Optimization #1 — Don’t Abuse Keys!**

> # Keys can turn your application into a rendering jackhammer.

![Keys. Potential Performance Killer.](https://cdn-images-1.medium.com/max/3440/1*mar5ltfc1QqQXv3TvFkK0A.png)<center>_Keys. Potential Performance Killer._</center>

Keys are an essential prop when iteratively rendering collections of elements. React uses them to keep track of elements in collections and poor choices for keys — keys that are not **stable, predictable and unique** — can lead to inefficient or even incorrect rendering behaviour. Examples of potentially suboptimal keys include timestamps (an unstable key that will always force a render), array indices (an unpredictable key that may not always map back to the same element) and duplicated keys.

![You might want to avoid errors that look like this.](https://cdn-images-1.medium.com/max/3120/1*ejyN3nQEkCJVxLaGnBRI8A.png)<center>_You might want to avoid errors that look like this._</center>

There is a time and a place to use keys (e.g. when dealing with lists) but it is also quite easy to **misuse** keys. Our data-reporting front-end had situations where keys were used to forcibly re-render non-iterated components in cases where render behaviour is poorly understood or when third party libraries are involved.

![The List isn’t rendering for some reason so let’s just force re-rendering with a key. Everything is awesome.](https://cdn-images-1.medium.com/max/4464/1*aXmvm_ytbP06V52i6gFVRw.png)<center>_The List isn’t rendering for some reason so let’s just force re-rendering with a key. Everything is awesome._</center>

As you may have realized, this particular pattern gave us serious performance headaches. The key prop makes further render optimizations impossible because it causes React to discard the entire component instance and DOM subtree every time the value of the key changes.

<br/>

## **Optimization #2 — Use PureComponent**

> # Use PureComponent when you have performance issues and have determined that a specific component was re-rendering too often, but its props and/or state are shallowly equal— Dan Abramov

The **PureComponent** base class was added to React in version 15.3.0 and provides a simple render optimization out of the box. A regular **Component** will render whenever it receives new props or state but a **PureComponent** will perform a shallow comparison of props and state and only render if there is a difference. The initial quote from Dan Abramov says it all — use PureComponent if you notice rendering happening even though props and state have not changed. These are avoidable renders and can be slowing your application down significantly!

I used [create-react-app](https://github.com/facebook/create-react-app) to scaffold a very simple demonstration of the impact of PureComponent [here](https://github.com/boweihan/react-performance-demo). The application renders a custom List component that iteratively generates a long list of custom ListItem components (20000 to be exact).

![List of 20000 ListItem Components](https://cdn-images-1.medium.com/max/3016/1*hGb2O_YDRNxoHos1y7sO1w.png)<center>_List of 20000 ListItem Components_</center>

The render button triggers a `setState` operation which simply passes back the same state. The performance of this application when simply extending **Component** is shown below. The colours in the flame graph indicate that all components are being re-rendered even though there is no difference in state and props.

![Application performance with List and ListItem as Components](https://cdn-images-1.medium.com/max/3664/1*DEGO9Ig_mSm0jUPDL6Lyww.png)<center>_Application performance with List and ListItem as Components_</center>

The render takes almost a second even though nothing new is applied to the page! Let’s now have ListItem extend **PureComponent** instead of **Component**. The ListItems now do not re-render (you can tell because they are greyed out).

![Application performance with ListItem as a PureComponent — 4x improvement!](https://cdn-images-1.medium.com/max/3648/1*frbTavo1dpdHAgww-XotgQ.png)<center>_Application performance with ListItem as a PureComponent — 4x improvement!_</center>

219 ms — getting there. Let’s now have List extend **PureComponent** as well. We can tell from the flame graph that the application is now smart enough to only re-render the app component after `setState`.

![Application performance with List and ListItem as PureComponents — 100x improvement!](https://cdn-images-1.medium.com/max/3652/1*vVD5AYXSkrayZRYi3MqJLw.png)<center>_Application performance with List and ListItem as PureComponents — 100x improvement!_</center>

Simply changing the base classes to be **PureComponent** and avoiding unnecessary re-renders was enough for a 100x performance improvement! This example may seem a tad trivial but re-rendering components when there are no changes to props or state is very common in applications that do not keep a keen eye on performance.

A great way to monitor your application for avoidable renders is to install an npm package called [why-did-you-update](https://github.com/maicki/why-did-you-update) which will monitor and log avoidable component renders straight into your browser console at runtime. My first use of this tool was an eye opener…

![Looks like there are more than a few wasted renders here :)](https://cdn-images-1.medium.com/max/2386/1*jzWmLzlU8_U1wjVEcfxZjA.png)<center>_Looks like there are more than a few wasted renders here :)_</center>

**PureComponent** is great when all you care about is shallow prop and state changes but there are cases when it doesn’t quite cut it — namely when…

1. Component render state is dependent on nested state/props.

1. State or props contains objects that are being re-created with the same properties (PureComponent uses === which compares **object references rather than actually comparing object properties** and since two different object instances will never satisfy strict equality the PureComponent will always re-render even if the object properties and values are the exact same!).

1. Certain properties on state or props are changing but do not affect the render state of the component (irrelevant props could be those that are passed through to child components).

Thankfully, we have options.

<br/>

## **Optimization #3 — ShouldComponentUpdate**

> # I’d just like to take a moment and thank #react for #shouldComponentUpdate ❤ — some person on twitter

**ShouldComponentUpdate** is a React component lifecycle method that determines whether or not a component should continue its render lifecycle. The method is called right before render. (Diagram below is straight from React docs).

![React LifeCycle Cheat Sheet](https://cdn-images-1.medium.com/max/3668/1*gMdgkSygxwy9mlJCFyphBg.png)<center>_React LifeCycle Cheat Sheet_</center>

**PureComponent** and **Component** actually implement this method under the hood in order to achieve selective rendering. I’ll start by providing some code samples which hopefully make it pretty clear what this method should do.

<br/>
### _Component_ implementation:

<br/>
<script src="https://gist.github.com/boweihan/9a952b8e9af279ad5bb77dc2eef417db.js"></script>
<br/>
### _PureComponent_ implementation:
<br/>
<script src="https://gist.github.com/boweihan/7d819eed40cf31ad4e276ca1611f7336.js"></script>
<br/>
The [shallowEqual implementation](https://github.com/facebook/fbjs/blob/master/packages/fbjs/src/core/shallowEqual.js) used in the above source code can be found on Facebook’s Github page.

ShouldComponentUpdate provides the flexibility for you to choose exactly **when** and **why** your components should update.

Back to Vena’s data-reporting application…a particularly poor area of performance involved a tree view component which we found was re-rendering entire subtrees instead of just the nodes affected by an update. The component is a tad complex and was producing a bloated flame graph for an operation which only updates the state of a single top-level node.

![](https://cdn-images-1.medium.com/max/4020/1*usZwjitmQ64LIPJtMNG3Bg.png)

This performance is acceptable for small trees but the user experience is unbearable for larger trees. The majority of these renders are avoidable. The data for each node is nested in nature so we can’t simply rely on **PureComponent** to sort out our render issues and instead have to implement a custom **shouldComponentUpdate** method.

<br/>
<script src="https://gist.github.com/boweihan/98c4a22049c71c3ecd5d5890ab1e81e0.js"></script>

<br/>
This change means that the nodes are now rendered on an as-needed basis and the component render count is effectively O(1) instead of O(N) with respect to the tree. The updated flame graph looks like this:

![](https://cdn-images-1.medium.com/max/3696/1*QZyKv1uESTRHKISn-8Qzag.png)

That’s a 328x improvement! ShouldComponentUpdate makes these optimizations possible.

## There’s more…

The three performance areas covered in this post — _keys, PureComponent, and ShouldComponentUpdate_ — are just the tip on the iceberg of potential performance improvements. We are conceptually solving caching problems and there are a few other potential areas for render optimizations that come to mind:

* Using [recompose](https://github.com/acdlite/recompose) to add selective rendering behaviour to your functional components.

* Memoizing computationally intensive electors with [reselect](https://github.com/reduxjs/reselect) (for Redux).

* Lazy loading components and virtualizing lists. Libraries such as [react-virtualized](https://github.com/bvaughn/react-virtualized) can help.

* Caching with service workers

* (insert your optimization here)

At Vena we were able to solve most of our immediate issues by tackling the highlighted areas but we’re ready for the next challenge as we continue to scale!

Comment on [Medium](https://medium.com/@BoweiHan/optimizing-react-rendering-61a10e741edb)
