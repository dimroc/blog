---
layout: post
title: "Not Quite Utopia: Golang and isomorphic React"
date: "Fri Feb 5 20:58:56 -0500 2016"
tags: go react
---

Below are the notes taken when developing my first Golang + React JS application:
[Cross City Search](http://urbanevents.dimroc.com/?q=tattoos).

Notes:

* Productivity just isn't there
* Boilerplate takes forever
  * Both the golang and react ecosystems are in flux.
  * What's best practice this year will be abandonware next year.
* Client siding routing with react-router is extremely time consuming
  * Gets even hard when you add redux, but redux-simple-router helps
  * Difficulty bumps up another level when you run into bugs rendering js server side.
  * Good luck debugging javascript rendered in duktape on golang.
* Isomorphic javascript starts off great, falls apart when half the client libraries you need require the `window` object
  * Bye bye bootstrap and a lot of other loved css frameworks.
* I love css modules and the ability to scope your styles.
* On the flipside, I have mixed feelings about coupling everything in a component together: view, style, and logic live in the component/my-component folder.
* Build configurations became pretty sophisticated despite Golang's amazing compiler
  * Used fancy cross compilation to prepare binaries for docker.
  * Had to roll that back for part of the app (cityweb) because we had native dependencies (go-duktape)
  * Webpack, the client side pipeline, was quiet a chore to set up, although there was a learning curve there.
  * Ended up using make files to create one liners to build go, build the docker image, and then push created image.

My top goal is to be productive, even if it's at the cost of peformance.
Golang and React JS is about as fast as it gets, but they lack the tooling and the frameworks
needed to make them competive in the realms of productivity.

My next project will be using the PEEP stack: Postgres, Elixir, Ember, Phoenix.
I hope this stack will hit the sweet spot of performance and productivity.
