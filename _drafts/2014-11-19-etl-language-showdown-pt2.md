---
layout: post
title: "Comparing Golang, Scala, Elixir and Ruby for ETL - Part 2"
date: "Wed Nov 19 10:56:59 -0500 2014"
tags: ruby elixir golang scala etl
---

Boy, do people love these language showdowns.

## Results
TODO

<!--more-->

### Ruby
Stayed as is.

### Scala
Stayed as is.

### Elixir
TODO: Talk about the HashDict issue.

Changing this

{% highlight elixir %}
Map.merge(...)
{% endhighlight %}

to this

{% highlight elixir %}
HashDict.merge(...)
{% endhighlight %}

made a dramatic difference! It speaks to the youth of the Elixir. That being said, this subtelty is being fixed
and won't catch unsuspecting programmers like myself again.

### Golang
TODO: Talk about Spawn method and using the channels as a thread-safe queue.

{% highlight go %}
// Spawns N routines, after each completes runs all whendone functions
func Spawn(N int, fn func(), whendone ...func()) {
  waiting := int32(N)
  for k := 0; k < N; k += 1 {
    go func() {
      fn()
      if atomic.AddInt32(&waiting, -1) == 0 {
        for _, fn := range whendone {
          fn()
        }
      }
    }()
  }
}
{% endhighlight %}

#####Usage

Note the channel `filenames` acting as a thread-safe queue.

{% highlight go %}
filenames := make(chan string, *procs)

Spawn(*procs, func() {
  for filename := range filenames {
    file, err := os.Open(filename)
    ...
}, func() { fmt.Println("Done") })
{% endhighlight %}

Thanks to all those who contributed to the repo:

- [Egon Elbre](https://github.com/egonelbre)
- [José Valim](https://github.com/josevalim)
- [gbulmer](https://github.com/gbulmer)

