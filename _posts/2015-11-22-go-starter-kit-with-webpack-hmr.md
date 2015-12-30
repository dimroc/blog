---
layout: post
title: "Boilerplate for the best of Go and React"
date: "Sun Nov 22 10:35:14 -0500 2015"
tags: go react
---

Want in on the killer combo of Golang with React? Check out the [go-starter-kit](https://github.com/olebedev/go-starter-kit).
It's a little rough around the edges, but once you get clean up that Makefile you'll have the following goodness:

1. [Hot Module Replace](http://webpack.github.io/docs/hot-module-replacement.html): _It's like live reload for every module._
2. Isomorphic React with [go-duktape](https://github.com/olebedev/go-duktape): Render HTML server side the first time, then client side every subsequent request.
3. [Redux](http://rackt.org/redux/): An evolution of the [Flux design pattern](https://facebook.github.io/flux/docs/overview.html) that's since been simplified for reloadability and development.
4. [Webpack dev server](https://github.com/webpack/webpack-dev-server) to hot reload while coding ala live reload off http://localhost:5001
5. One terminal window process for all development!

I did have to patch up the Makefile to support my usage of the [recommended package paths](https://golang.org/doc/code.html#PackagePaths).

ie: `$GOPATH/src/github.com/user/your_app`

### Makefile

{% highlight makefile %}
BIN = $(GOPATH)/bin
NODE_BIN = $(shell npm bin)
PID = .pid
GO_FILES = $(filter-out app/server/bindata.go, $(shell find app -type f -name "*.go"))
BINDATA = app/server/bindata.go
BINDATA_FLAGS = -pkg=server -prefix=app/server/data
BUNDLE = app/server/data/static/build/bundle.js
APP = $(shell find app/client -type f)
GO_APP = github.com/dimroc/urbanevents/cityweb/app # Addition: Introduced GO_APP path

build: clean $(BIN)/app

clean:
	@rm -rf app/server/data/static/build/*
	@rm -rf app/server/data/bundle.server.js
	@rm -rf $(BINDATA)
	@echo cleaned

$(BUNDLE): $(APP)
	@$(NODE_BIN)/webpack --progress --colors

$(BIN)/app: $(BUNDLE) $(BINDATA)
	@go install $(GO_APP) # Swapped in GO_APP instead of app

kill:
	@kill `cat $(PID)` || true

serve: clean $(BUNDLE)
	@make restart
	@$(NODE_BIN)/webpack-dev-server --config webpack.hot.config.js $$! > $(PID)_wds &
	@ANYBAR_WEBPACK=yep $(NODE_BIN)/webpack --progress --colors --watch $$! > $(PID)_wp &
	@fswatch $(GO_FILES) | xargs -n1 -I{} make restart || make kill
	@kill `cat $(PID)_wp` || true
	@kill `cat $(PID)_wds` || true

restart: BINDATA_FLAGS += -debug
restart: $(BINDATA)
	@make kill
	@go install $(GO_APP) # Swapped in GO_APP instead of app
	@$(BIN)/app run & echo $$! > $(PID)

$(BINDATA):
	$(BIN)/go-bindata $(BINDATA_FLAGS) -o=$@ app/server/data/...

lint:
	@eslint app/client || true
	@golint $(filter-out app/main.go, $(GO_FILES)) || true
	@golint -min_confidence=1 app

{% endhighlight %}
