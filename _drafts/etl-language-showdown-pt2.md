---
layout: post
title: "Comparing Golang, Scala, Elixir, Ruby, and now Python3 for ETL: Part 2"
date: "Thu May 7 11:13:39 -0500 2015"
tags: ruby elixir golang scala python etl
---

A year ago, I wrote the same program in four languages to compare their productivity when performing ETL (extract-transform-load).
Read about [part 1 here](/2014/09/29/etl-language-showdown/) and check out the [source code](https://github.com/dimroc/etl-language-comparison).

The code has changed, the languages have evolved, and the hardware now includes a SSD drive. Where are they now?

<!--more-->

## Results

<table>
  <tr>
    <td>Ruby w/ Celluloid (Global Interpreter Lock Bound, single core)</td>
    <td>43.7s</td>
  </tr>

  <tr>
    <td>JRuby w/ Celluloid</td>
    <td>15.8s</td>
  </tr>

  <tr>
    <td>Ruby w/ <a href="https://github.com/grosser/parallel" target="_blank">grosser/parallel</a> (<b>not</b> GNU Parallel)</td>
    <td>10.9s</td>
  </tr>

  <tr>
    <td>Python w/ <a href="https://docs.python.org/2/library/multiprocessing.html" target="_blank">Pool</a></td>
    <td>12.7s</td>
  </tr>

  <tr>
    <td>Scala</td>
    <td>8.8s</td>
  </tr>

  <tr>
    <td>Scala w/ Substring <b>(Skipped regex for performance analysis)</b></td>
    <td>8.3s</td>
  </tr>

  <tr>
    <td>Golang</td>
    <td>32.8s</td>
  </tr>

  <tr>
    <td>Golang w/ Substring <b>(Skipped regex for performance analysis)</b></td>
    <td>7.8s</td>
  </tr>

  <tr>
    <td>Elixir</td>
    <td>21.8s</td>
  </tr>
</table>

## Recap

The original goal was **not** to see how fast each language could go, it was was to measure the length of time needed to
write a solution in that language, and to subjectively measure the maintainability of that final solution learning about the gotchas on the way.
But in the end everyone wants benchmarks.

It was assumed that runtimes would all be approximately the same, since this should have been an IO-bound problem. So why
care about the speed of the language? Well, on my old MacBook Pro with a 5200 RPM HDD, this was not true. Is it true on my SSD?
It still isn't.

## The Hardware

MacBook Pro 2.3GHz i7 (quad core) with 16GB RAM and SSD

## The Problem

We have ~40M tweets spanning multiple files, with each tweet tagged with their New York City neighborhood. Discover which
neighborhoods care the most about the New York Knicks by searching for the term `knicks`.

## Questions and Concerns from [Part 1](/2014/09/29/etl-language-showdown/)

1. Why am I writing to an intermediary file? Why don't I do it all in memory? Now I do.

    This comparison was derived from a larger ETL process that spanned multiple computers and therefore
    used intermediary files to pass along the information. This cookie-cutter experiment has no need for this,
    so it has been removed.

2. Why am I using regex and not a simple string search (GoLang's regex sucks in 1.x.x)?

    The implementations should be consistent across all languages for a fair comparison. Although
    the problem is simply searching for `knicks`, I wanted the implementations to have the flexibility to
    to perform more powerful searches. That being said, Golang's Regexp package performs dramatically worse than other languages.

3. In Scala, why did I use Akka instead of the lighter Parallel Collections?

    Because I love Akka.

## Implementation Changes

### Ruby
- Ruby version is now 2.2.2.
- No longer uses GNU Parallel, but instead uses [grosser/parallel](https://github.com/grosser/parallel) to span multiple cores.
- Implementation no longer writes to intermediary file.

### Scala
- Upgraded to Scala 2.11.5 and Akka 2.3.10.
- Reduction no longer writes to intermediary file.
- Still uses Akka. If you think the [Parallel Collections](http://docs.scala-lang.org/overviews/parallel-collections/overview.html) library would be a better fit,
which it very well might be, please feel free to contribute a pull request.

### Python
- Version python3-3.4.3
- A new Python implementation has been added for comparison's sake.
- The [`Pool` object](https://docs.python.org/2/library/multiprocessing.html) allows one to run the program on multiple processes and sidestep the Global Interpreter Lock (GIL).
    A pretty great alternative to my use of [GNU `parallel`](http://www.gnu.org/software/parallel/) with Ruby in [part 1](/2014/09/29/etl-language-showdown/).

### Elixir
- Updated to Elixir version 1.0.4
- Reduction no longer writes to intermediary file.
- Actor model is beautiful in Elixir.
- No significant performance improvement when using String.contains instead of regex.
- Profiled with [exprof](https://github.com/parroty/exprof) but didn't see any low hanging fruit (I'm welcome to any feedback here).
    ![Elixir Profiling](/public/images/etlElixirProfiling.jpg)
- Changing this

{% highlight elixir %}
Map.merge(...)
{% endhighlight %}

to this

{% highlight elixir %}
HashDict.merge(...)
{% endhighlight %}

made a dramatic difference. It speaks to the youth of the Elixir. That being said, this subtelty is being fixed
and won't catch unsuspecting programmers like myself again.

[From the website:](http://elixir-lang.org/getting-started/maps-and-dicts.html#maps)

> Note: Maps were recently introduced into the Erlang VM with EEP 43. Erlang 17 provides a partial implementation of the EEP, where only "small maps" are supported. This means maps have good performance characteristics only when storing at maximum a couple of dozens keys. To fill in this gap, Elixir also provides the HashDict module which uses a hashing algorithm to provide a dictionary that supports hundreds of thousands keys with good performance.

### Golang

- Updated Golang to 1.4.2.
- Initial performance was a disappointing 30s+, so I dug in and used [pprof](http://blog.golang.org/profiling-go-programs) to profile the code.
    ![Golang Profiling](/public/images/etlGolangRegexp.jpg)
- Go's Regular Expression engine really is as slow as a previous commenter mentioned. Switching to `strings.Contains` took it to ~7s.
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

- It's always a challenge (or a lot of fun) attempting to write the same thing in two languages, let alone five.
A langauge's idioms sway an implementation in a particular direction. Long story short, there are still a lot of discrepancies between the implementations.
- Elixir and Golang have matured dramatically in a year's time.
- It is damn difficult to parse Scala code when you've been away for a while. It's just **dense**.
- [My previous conclusion](/2014/09/29/etl-language-showdown/) still holds up, check it out (it's at the bottom of the link).
- This whole experiment has lived far longer than I thought.

## Think you can do better? Want to see another language? Contribute.

Submit a pull request with your code changes and I'll update the doc.

## Thanks

Thanks to all those who contributed to the repo:

- [Egon Elbre](https://github.com/egonelbre) - Go
- [gbulmer](https://github.com/gbulmer) - Go
- [José Valim](https://github.com/josevalim) - Elixir
- [Aaron Held](https://github.com/aheld) - Python

