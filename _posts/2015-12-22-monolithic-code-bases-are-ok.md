---
layout: post
title: "Many Services Doesn't Mean Many Repos: Monolithic Code Bases Are OK"
date: "Tue Dec 22 19:17:02 -0500 2015"
tags: architecture
---

There's been a recent obsession with microservices.

Apparently everyone's scaling to meet crazy demand (congrats),
and the claim is that their slow fat monolithic application is holding them back.

Unfortunately people also assume that making many services means they
have to create many repositories.

## Separate Repos Are Much Harder to Maintain

Over time, I have come to advocate for a large single code repository that has multiple entry points.

* Easier to test.
* Easier to version.
* Easier dependency management.

### Origins of the Complaint

It stems from the common Rails scenario where a bloated application has horrendous startup time and
poor code isolation.

Sidekiq or Resque Jobs adhering to a simple contract (`def perform`) and living in a `jobs/` folder is a simple example where a single code base happily
hosts two services: the web server, and the background worker. And as a result, testing, versioning,
and deployments are significantly easier.

Imagine the overhead in testing, maintaining, and synchronizing your application when the web server and
the background workers live in a separate code base.

* Each repo has to be spun up with seed data for true integration testing.
* You could build and maintain [fake versions of your service](/2014/09/30/toggleable-fakes).
* Manual testing in a QA environment.

It's a lot easier just running tests inline in a single code base. And there are tools to help with this (Protocol Buffers, etc).

### Golang's philosophy

I'm not the only one who thinks this. Google [has already stated they lean towards large code bases](http://www.wired.com/2015/09/google-2-billion-lines-codeand-one-place/).
Which is why it's no surprise that Golang was designed for blazing fast compile times even
in the face of large code bases. Most golang code bases have a Command Line Interface (CLI) to
run different aspects of the application.

The same is also true for MS Office code. It's housed in a repo so large, they have to use Razzle source browser
instead of Visual TFS to navigate. At least as of 2011.

### It's Not A Silver Bullet

It's just unfortunate that Ruby and Rails isn't better at being large. Ruby's poor namespacing pisses me off.
But it does so much so well, it's hard to hold that against it.

Loading the whole environment, as advocated by the default Rails setup, can hold you back, but that doesn't always mean separate repos.
It's gotten you this far though.

Perhaps separate repos are the answer, but don't jump to it as quick as others have. You're not Twitter (yet).

