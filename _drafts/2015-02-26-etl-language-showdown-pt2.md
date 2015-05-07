---
layout: post
title: "RE: Comparing Golang, Scala, Elixir, Ruby, and now Python3 for ETL"
date: "Thu Feb 26 11:13:39 -0500 2015"
tags: ruby elixir golang scala python etl
---

A year ago, I wrote the same program in four languages to compare their productivity performing ETL (extract-transform-load).
Read about [part 1 here](/2014/09/29/etl-language-showdown/) and check out the [source code](https://github.com/dimroc/etl-language-comparison).

The code has changed, the languages have evolved, and the hardware now includes a SSD drive. Where are they now??

## Results

<table>
  <tr>
    <td>Ruby (Global Interpreter Lock bound)</td>
    <td>1m32.624s</td>
  </tr>

  <tr>
    <td>Ruby w/ GNU Parallel</td>
    <td>45.803s</td>
  </tr>

  <tr>
    <td>Python w/ Pool</td>
    <td>11.532s</td>
  </tr>

  <tr>
    <td>Scala</td>
    <td>27.247s</td>
  </tr>

  <tr>
    <td>Golang</td>
    <td>48.556s</td>
  </tr>

  <tr>
    <td>Elixir</td>
    <td>58.907s</td>
  </tr>
</table>

<!--more-->

## The Hardware

MacBook Pro 2.3GHz i7 (quad core) with 16GB RAM and SSD

## The Problem

We have ~40M tweets spanning multiple files, with each tweet tagged with their New York City neighborhood. Discover which
neighborhoods care the most about the New York Knicks by searching for the term `knicks`.

## Recap

The goal was **not** to see how fast each language could go. The goal was to measure the length of time needed to
write a solution in that language, and to subjectively measure the maintainability of the final solution.

It was assumed that runtimes would all be approximately the same, since this should have been an IO-bound problem. So why
care about the speed of the language? Well, on my old MacBook Pro with a 5200 RPM HDD, this was not true. Is it true on my SSD??? TODO:wjioj4iovaklsd

## Questions and Concerns from Part 1

1. Why am I writing to an intermediary file? Why don't I do it all in memory?

    This comparison was derived from a larger ETL process that spanned multiple computers and therefore
    used intermediary files to pass along the information. This cookie-cutter experiment has no need of this,
    so it's been removed.

2. Why am I using regex and not a simple string search (GoLang's regex sucks in 1.x.x)?

    The implementations should be consistent across all languages for a fair comparison. Although
    the problem is simply searching for `knicks`, I wanted the implementations to have the flexibility to
    to perform more powerful searches.

3. In Scala, why did I use Akka instead of the lighter Parallel Collections?

    Because I love Akka and wanted to check it out.

## Implementation Changes

### Ruby
Ruby version is now 2.2.0. Implementation stayed as is.

### Scala
Stayed as is.

### Python
- Version python3-3.4.3
- A new Python implementation has been added for comparison's sake.
- The whole thing is 64 lines, gotta love its terseness. This isn't a shortness competition though so please don't get carried away with the lines of code metric.
- The [`Pool` object](https://docs.python.org/2/library/multiprocessing.html) allows one to run the program on multiple processes and sidestep the Global Interpreter Lock (GIL).
    A pretty great alternative to my use of [GNU `parallel`](http://www.gnu.org/software/parallel/) with Ruby.

### Elixir
- Updated to Elixir version 1.0.3
- Changing this

{% highlight elixir %}
Map.merge(...)
{% endhighlight %}

to this

{% highlight elixir %}
HashDict.merge(...)
{% endhighlight %}

made a dramatic difference! It speaks to the youth of the Elixir. That being said, this subtelty is being fixed
and won't catch unsuspecting programmers like myself again.

[From the website:](http://elixir-lang.org/getting-started/maps-and-dicts.html#maps)

> Note: Maps were recently introduced into the Erlang VM with EEP 43. Erlang 17 provides a partial implementation of the EEP, where only "small maps" are supported. This means maps have good performance characteristics only when storing at maximum a couple of dozens keys. To fill in this gap, Elixir also provides the HashDict module which uses a hashing algorithm to provide a dictionary that supports hundreds of thousands keys with good performance.

### Golang

- Updated Golang to 1.4.2.
- They've been hyped before, but I'm going to hype them again: GoLang's Channels are fantastic.

A modification to the GoLang implementation liberally uses channels as a FIFO queue to great effect:

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

## Conclusion

- There's a lot of knowledge here. I definitely got what I wanted out of this comparison.
- It's always a challenge (or a lot of fun) attempting to write the same thing in two languages, let alone five. A langauge's idioms sway an implementation in a particular direction. Long story short, there are a lot of discrepancies between the implementations.
- Elixir and Golang have matured dramatically in a year's time.
- [My previous conclusion](/2014/09/29/etl-language-showdown/) still holds up, check it out (it's at the bottom).
- This whole experiment has lived far longer than I thought.

## Thanks

Thanks to all those who contributed to the repo:

- [Egon Elbre](https://github.com/egonelbre) - Go
- [gbulmer](https://github.com/gbulmer) - Go
- [José Valim](https://github.com/josevalim) - Elixir
- [Aaron Held](https://github.com/aheld) - Python

