---
layout: post
title: "tutum exec to replace heroku run"
date: "Sun Jan 10 13:56:40 -0500 2016"
tags: tutum docker elixir
---

`heroku run rails console` is an amazing commandline utility enabling one to run a REPL loop on their production server.

Docker comes with an `exec` command that allows you to ssh into a running container much like the `heroku run` command.
But doing this on a remotely running docker container can be a hassle.

This is now much easier via docker with tutum. See this example with elixir:

![Tutum Exec](/public/images/tutumExec.jpg)
