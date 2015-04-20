---
layout: post
title: "amazon-kinesis-for-autoscaling-work"
date: "Mon Apr 20 08:38:32 -0700 2015"
---

## TODO

Many hours have been spent optimizing the cost of a backend. Iron.io has Iron Worker that scales up when a job is submitted,
Sidekiq has HireFire to spin up and down Heroku Dynos and AWS has Autoscale groups for EC2 workers.

But now there's Amazon Kinesis, and that changes everything!
