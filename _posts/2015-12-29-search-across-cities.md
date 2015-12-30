---
layout: post
title: "Urban Events: Compare NYC media against LA and other cities"
date: "Tue Dec 29 18:52:35 -0500 2015"
tags: go react elasticsearch docker
---

Just released [Urban Events](http://urbanevents.dimroc.com/?q=tattoos)!

![Urban Events](/public/images/UrbanEvents.jpg)

* This experiment allows one to search for media across cities and neighborhoods.
* The technology used was Golang, React JS, Webpack, Elasticsearch, and Docker.


### High Level Services

1. **City Recorder**: Classifies tweets from Twitter's [Public Streaming API](https://dev.twitter.com/streaming/reference/post/statuses/filter)
with a neighborhood by running Elasticsearch (ES) Geospatial percolations against an index of
[city neighborhood GeoJSON files](https://github.com/dimroc/neighborhoods).
2. **City Web**: Searches across cities using Elasticsearch's `top_hits` metric aggregator and displays results in React JS.

Currently listening to NYC, London, Paris, Austin, Miami, and Los Angeles.

Check out the [github page](https://github.com/dimroc/urbanevents) for technical details and source code.

