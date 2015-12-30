---
layout: post
title: "not-quite-utopia-golang-isomorphic-react"
date: "Mon Dec 14 20:58:56 -0500 2015"
tags: go react
---

Notes:

* ##### COINCIDE WITH CROSS CITY SEARCH RELEASE #####
* Productivity just isn't there
* Boilerplate takes forever
* Client siding routing with react-router is extremely time consuming
* Gets even hard when you add redux, but redux-simple-router helps
* Difficulty bumps up another level when you run into bugs rendering js server side.
    Good luck debugging javascript rendered in duktape on golang.
* I love css modules and the ability to scope your styles.
* On the flipside, I have mixed feelings about coupling everything in a component together: view, style, and logic live in the component/my-component folder.
* Build configurations became pretty sophisticated despite Golang's amazing compiler
  * Used fancy cross compilation to prepare binaries for docker.
  * Had to roll that back for part of the app (cityweb) because we had native dependencies (go-duktape)
  * Webpack, the client side pipeline, was quiet a chore to set up, although there was a learning curve there.
  * Ended up using make files to create one liners to build go, build the docker image, and then push created image.


Conclusion:

Enjoyed ramping up on it, but decided to abandon the React SPA in favor 
of some more server side rendering. Routing is simply too difficult and the isomorphic rendering isn't
worth it when bootstrapping a site.
