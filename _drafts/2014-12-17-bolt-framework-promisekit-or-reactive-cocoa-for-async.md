---
layout: post
title: "Taming Callbacks in iOS: Bolts Framework"
date: "Wed Dec 17 18:53:26 -0500 2014"
tags: ios swift
---

Managing many asynchronous events can get hairy, but it's a fact of life when working with
remote services. So how do we avoid callback hell?

**Promises (aka futures)**.

Below I'll explain why I chose the [**Bolts Framework (BF)**](https://github.com/BoltsFramework/Bolts-iOS)
over [PromiseKit](https://github.com/mxcl/PromiseKit) and [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa).

<!--more-->
