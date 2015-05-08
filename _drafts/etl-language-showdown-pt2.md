---
layout: post
title: "Comparing Golang, Scala, Elixir, Ruby, and now Python3 for ETL: Part 2"
date: "Thu May 7 11:13:39 -0500 2015"
tags: ruby elixir golang scala python etl
---

A year ago, I wrote the same program in four languages to compare their productivity when performing ETL (extract-transform-load).
Read about [part 1 here](/2014/09/29/etl-language-showdown/) and check out the [source code](https://github.com/dimroc/etl-language-comparison).

The code has changed, the languages have evolved, and the hardware now includes a SSD drive. Where are they now?

## Recap

The goal was **not** to see how fast each language could go. The goal was to measure the length of time needed to
write a solution in that language, and to subjectively measure the maintainability of the final solution.
But in the end everyone wants benchmarks, so those are provided.

It was assumed that runtimes would all be approximately the same, since this should have been an IO-bound problem. So why
care about the speed of the language? Well, on my old MacBook Pro with a 5200 RPM HDD, this was not true. Is it true on my SSD?
It still isn't. A max of a day was spent on each solution, so time for optimizations was capped.

## Results

<table>
  <tr>
    <td>Ruby w/ Celluloid (Global Interpreter Lock Bound, single core)</td>
    <td>43.745s</td>
  </tr>

  <tr>
    <td>JRuby w/ Celluloid</td>
    <td>15.855s</td>
  </tr>

  <tr>
    <td>Ruby w/ <a href="https://github.com/grosser/parallel" target="_blank">grosser/parallel</a> (<b>not</b> GNU Parallel)</td>
    <td>12.449s</td>
  </tr>

  <tr>
    <td>Python w/ <a href="https://docs.python.org/2/library/multiprocessing.html" target="_blank">Pool</a></td>
    <td>12.730s</td>
  </tr>

  <tr>
    <td>Scala</td>
    <td>8.8s</td>
  </tr>

  <tr>
    <td>Golang</td>
    <td>TODO</td>
  </tr>

  <tr>
    <td>Elixir</td>
    <td>21.84s</td>
  </tr>
</table>

<!--more-->

## The Hardware

MacBook Pro 2.3GHz i7 (quad core) with 16GB RAM and SSD

## The Problem

We have ~40M tweets spanning multiple files, with each tweet tagged with their New York City neighborhood. Discover which
neighborhoods care the most about the New York Knicks by searching for the term `knicks`.

## Questions and Concerns from Part 1

1. Why am I writing to an intermediary file? Why don't I do it all in memory?

    This comparison was derived from a larger ETL process that spanned multiple computers and therefore
    used intermediary files to pass along the information. This cookie-cutter experiment has no need for this,
    so it's been removed.

2. Why am I using regex and not a simple string search (GoLang's regex sucks in 1.x.x)?

    The implementations should be consistent across all languages for a fair comparison. Although
    the problem is simply searching for `knicks`, I wanted the implementations to have the flexibility to
    to perform more powerful searches.

3. In Scala, why did I use Akka instead of the lighter Parallel Collections?

    Because I love Akka and wanted to check it out.

## Implementation Changes

### Ruby
- Ruby version is now 2.2.1.
- No longer uses GNU Parallel, but instead uses [grosser/parallel](https://github.com/grosser/parallel) to span multiple cores
- Implementation no longer writes to intermediary file.

### Scala
- Upgraded to Scala 2.11.5 and Akka 2.3.10.
- Reduction no longer writes to intermediary file.
- Still uses Akka. If you think the [Parallel Collections](http://docs.scala-lang.org/overviews/parallel-collections/overview.html) library would be a better fit,
which it very well might be, please feel free to contribute a pull request.

### Python
- Version python3-3.4.3
- A new Python implementation has been added for comparison's sake.
- The whole thing is 64 lines, gotta love its terseness. This isn't a shortness competition though so please don't get carried away with the lines of code metric.
- The [`Pool` object](https://docs.python.org/2/library/multiprocessing.html) allows one to run the program on multiple processes and sidestep the Global Interpreter Lock (GIL).
    A pretty great alternative to my use of [GNU `parallel`](http://www.gnu.org/software/parallel/) with Ruby.

### Elixir
- Updated to Elixir version 1.0.4
- Reduction no longer writes to intermediary file.
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
- It's always a challenge (or a lot of fun) attempting to write the same thing in two languages, let alone five.
A langauge's idioms sway an implementation in a particular direction. Long story short, there are a lot of discrepancies between the implementations.
- Elixir and Golang have matured dramatically in a year's time.
- It is damn difficult to parse Scala code when you've been away for a while. It's just **dense**.
- [My previous conclusion](/2014/09/29/etl-language-showdown/) still holds up, check it out (it's at the bottom).
- This whole experiment has lived far longer than I thought.

## Think you can do better? Contribute.

Submit a pull request with your code changes and I'll update the doc!

## Thanks

Thanks to all those who contributed to the repo:

- [Egon Elbre](https://github.com/egonelbre) - Go
- [gbulmer](https://github.com/gbulmer) - Go
- [José Valim](https://github.com/josevalim) - Elixir
- [Aaron Held](https://github.com/aheld) - Python

