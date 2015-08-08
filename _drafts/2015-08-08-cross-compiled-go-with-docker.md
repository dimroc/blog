---
layout: post
title: "cross-compiled-go-with-docker"
date: "Sat Aug 08 10:46:35 -0400 2015"
tags: go docker
---

Cross compilation is amazing! Release self-contained binary with no dependencies.

Coupled with a minimalist docker, alpine, and you have a tiny 40MB image. As opposed to a 500MB image
when you have to compile the go code when building the Dockerfile.
