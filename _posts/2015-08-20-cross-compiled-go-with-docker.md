---
layout: post
title: "Cross Compiled Go with Alpine Linux make a tiny Docker image"
date: "Sat Aug 20 10:46:35 -0400 2015"
tags: go docker
---

Golang's ability to create a self-contained executable makes deployment a breeze.
You just copy the single file. No need to worry about versioned dependencies and your dependencies' dependencies.

Golang's compiler goes ever further by supporting [cross compilation](http://stackoverflow.com/questions/12168873/cross-compile-go-on-osx).
Know your target architecture?
Compile directly for it from your dev box. Linux example below:

{% highlight bash %}
brew install go --with-cc-common # Installs go with cross compilation support
GOOS=linux GOARCH=amd64 go build -o someexecutable
{% endhighlight %}

Couple this with a minimalist docker image, such as [alpine](https://github.com/gliderlabs/docker-alpine), and you have yourself a tiny 40MB image.

#### Dockerfile
{% highlight bash %}
FROM alpine:3.2
ADD someexecutable /go/bin/someexecutable
ENTRYPOINT /go/bin/someexecutable
{% endhighlight %}

Sometimes you'll need an extra thing or two, like CA certificates to connect to HTTPS with SSL:

#### Dockerfile
{% highlight bash %}
FROM alpine:3.2
RUN apk add --update ca-certificates # Certificates for SSL
ADD tmp/someexecutable /go/bin/someexecutable
ENTRYPOINT /go/bin/someexecutable
{% endhighlight %}

Compare this to my original 500MB image built from the [golang base image](https://github.com/docker-library/golang):

#### Dockerfile
{% highlight bash %}
# Start from a Debian image with the latest version of Go installed
# and a workspace (GOPATH) configured at /go.
FROM golang # This line alone will put you at about 300MB.

# Copy the local package files to the container's workspace.
ADD . /go/src/github.com/dimroc/urbanevents/cityservice

# Build the outyet command inside the container.
# (You may fetch or manage dependencies here,
# either manually or with a tool like "godep".)

WORKDIR /go/src/github.com/dimroc/urbanevents/cityservice

RUN wget https://raw.githubusercontent.com/pote/gpm/v1.3.2/bin/gpm && \
      chmod +x gpm && \
      mv gpm /usr/local/bin # GPM

RUN gpm
RUN go install github.com/dimroc/urbanevents/cityservice/cityrecorder
WORKDIR /go/src/github.com/dimroc/urbanevents/cityservice/cityrecorder
ENTRYPOINT /go/bin/cityrecorder
{% endhighlight %}


With the new minimalist Dockerfile, pushing incremental changes only sends
a trivial 11MB and the image is now a reasonable 40MB. A far cry from the hundred MB sized docker
images floating around.

Cross Compiled Go + Alpine Linux + Docker = Win
