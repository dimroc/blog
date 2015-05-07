---
layout: post
title: "Amazon Lambda to Autoscale Background Work"
date: "Mon Apr 20 08:38:32 -0700 2015"
---

Many hours have been spent optimizing the cost background work. [Iron.io](http://www.iron.io/) has Iron Worker that scales up when a job is submitted,
[HireFire](https://www.hirefire.io/) spins up and down Heroku Dynos and AWS has Autoscale groups for EC2 instances.

Now [Amazon Lambda's](http://aws.amazon.com/lambda/) joining the party. Only supporting node.js for the time being, this service magically autoscales
based on size of whatever queue is feeding it. It responds to a new entry into an empty queue within milliseconds and is
extremely cost efficient.

A part of me feels for the hundreds of companies that rolled out their own scalable backend only to see this deprecate all that work overnight.
I would know, I worked on one.
