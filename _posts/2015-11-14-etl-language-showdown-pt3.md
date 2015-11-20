---
layout: post
title: "ETL Language Showdown Part 3"
date: "Nov 14 10:52:19 -0400 2015"
---

_This article is a continuation of the [Extract-Transform-Load (ETL)](https://en.wikipedia.org/wiki/Extract,_transform,_load) showdown series.
Check these posts out for additional context and check out the [repo](https://github.com/dimroc/etl-language-comparison)._

* _[Comparing Golang, Scala, Elixir and Ruby for ETL](/2014/09/29/etl-language-showdown/)_
* _[Comparing Golang, Scala, Elixir, Ruby, and now Python3 for ETL: Part 2](/2015/05/07/etl-language-showdown-pt2)_

In this post, we will compare a Map Reduce solution to count the number of times `knicks`
is mentioned in ~40M tweets spanning multiple files. This originally came about as a way to compare vanilla
[GIL-bound](https://en.wikipedia.org/wiki/Global_interpreter_lock) Ruby implementations against Scala
and Golang. It has since evolved into a [repo of idiomatic ETL solutions](https://github.com/dimroc/etl-language-comparison)
for a variety of languages (10 at the time of this writing).

## Shortcuts led to apple and orange comparisons

During this adventure, my Golang solution wasn't as performant as I had hoped because the regex
library couldn't compete with Scala's. As a result, I introduced the first apple to orange comparison by allowing Golang to use substrings.

One can't help but find the fastest solution possible, and this story continued with other languages and pull requests.
Reference implementations were accompanied with optimized solutions taking shortcuts, and I couldn't turn away such sweet code.

Even when sticking to the reference solution, it has becoming increasingly difficult to keep the implementations consistent across languages
because certain idioms sway solutions in different directions.

### Examples of shortcuts

1. PHP takes a shortcut by using `stripos` that only works on ASCII, not Unicode.
2. Regex comparison as opposed to substring checks.
3. Loading file contents into memory as opposed to streaming.
4. Having the master or supervisor keep track of the results so there is effectively no reduction step.
5. Leveraging binary pattern matching as opposed to regular regex.

Here are the previous implementations described in [part 2 of the ETL showdown](/2015/05/07/etl-language-showdown-pt2/):

<div height="800px">
  <canvas id="languageChart"></canvas>
</div>

# Takeaway

The results have a healthy amount of variety. Going into [part 1](/2014/09/29/etl-language-showdown/), it was believed that
the results of multi-threaded approaches would be the same because the problem would be IO bound. Well, that was wrong.
Instead of using this as a speed comparison across languages, use this repo as a reference for writing idiomatic ETL solutions in the language of your choice.
[Check out the repo.](https://github.com/dimroc/etl-language-comparison)

In order to promote consistency across implementations, I've introduced the following:

### Rules of Reference Implementation

1. Stream input from files.
2. Use Regular Expressions to check for the presence of `knicks`.
3. Have multiple mappers, but one reducer.
4. Each individual worker holds its results in a hash and sends that final hash back for reduction.

### New School: Language Additions

<div height="800px">
  <canvas id="languageChart2"></canvas>
</div>

## New School Observations and Shortcuts

We've since added even more languages, each with their own nuances.
[Nim](http://nim-lang.org/) is a newcomer I've never heard of but it performed surprisingly well.
Below you'll find some notes on each implementation.

### [Erlang](https://github.com/dimroc/etl-language-comparison/tree/master/erlang)

1. Leverages [Binary Pattern Matching](http://www.erlang.org/doc/efficiency_guide/binaryhandling.html).
2. Holds all data in memory as opposed to streaming input line by line.

### [Nim](https://github.com/dimroc/etl-language-comparison/tree/master/nim)

1. Implementation seems identical to the ruby version (streaming input with regex but using multiple cores) and it is blazing fast. The first I've heard of [Nim](http://nim-lang.org/).

### [Rust](https://github.com/dimroc/etl-language-comparison/tree/master/rust)

1. Regex solution is identical to reference implementation.
2. Substring solution only deals with ASCII.
3. Man is it fast. Would love someone to take another look at the Rust implementation to see how it might be taking shortcuts against the reference implementation.

### [PHP](https://github.com/dimroc/etl-language-comparison/tree/master/php)

1. Only doing substring checks against ASCII.
2. Single threaded.
3. Included just to grow the library of implementations. Improvements and pull requests are welcome.

### [Node JS](https://github.com/dimroc/etl-language-comparison/tree/master/nodejs)

1. Uses [cluster](https://nodejs.org/api/cluster.html) to get around the [Global Interpreter Lock (GIL)](https://en.wikipedia.org/wiki/Global_interpreter_lock).
2. The workers communicate a match to the cluster master, effectively skipping the reduction step.


## Call To Action

* The number of languages being covered has grown past my ability to maintain. If you would like to **own** a particular language's implementation,
  please let me know and feel free to submit PR's my way.
* Another area of interest is memory consumption. Tracking memory consumption can be tricky.
  For example, runtimes can aggresssively grab memory despite the application not using all of it. It would
  be great to track peak memory usage for the runs, so that we can now plot memory consumption along with execution time.

### Langauges Covered

<table>
<tr> <th>Language</th><th>Owner</th> </tr>
<tr> <td><a href="https://github.com/dimroc/etl-language-comparison/tree/master/ruby">Ruby</a></td><td> </td> </tr>
<tr> <td><a href="https://github.com/dimroc/etl-language-comparison/tree/master/golang">Golang</a></td><td></td> </tr>
<tr> <td><a href="https://github.com/dimroc/etl-language-comparison/tree/master/scala">Scala</a></td><td></td> </tr>
<tr> <td><a href="https://github.com/dimroc/etl-language-comparison/tree/master/nim">Nim</a></td><td></td> </tr>
<tr> <td><a href="https://github.com/dimroc/etl-language-comparison/tree/master/nodejs">Node</a></td><td></td> </tr>
<tr> <td><a href="https://github.com/dimroc/etl-language-comparison/tree/master/php">PHP</a></td><td></td> </tr>
<tr> <td><a href="https://github.com/dimroc/etl-language-comparison/tree/master/erlang">Erlang</a></td><td><a href="https://github.com/potatosalad">potatosalad</a></td> </tr>
<tr> <td><a href="https://github.com/dimroc/etl-language-comparison/tree/master/elixir">Elixir</a></td><td><a href="https://github.com/josevalim">josevalim</a></td> </tr>
<tr> <td><a href="https://github.com/dimroc/etl-language-comparison/tree/master/rust">Rust</a></td><td><a href="https://github.com/potatosalad">potatosalad</a></td> </tr>
<tr> <td><a href="https://github.com/dimroc/etl-language-comparison/tree/master/python">Python</a></td><td></td> </tr>
<tr> <td><a href="https://github.com/dimroc/etl-language-comparison/tree/master/csharp">C#</a></td><td><a href="https://github.com/mganss">mganss</a></td> </tr>
<tr> <td><a href="https://github.com/dimroc/etl-language-comparison/tree/master/shell">shell</a></td><td><a href="https://github.com/mganss">mganss</a></td> </tr>
<tr> <td><a href="https://github.com/dimroc/etl-language-comparison/tree/master/perl">perl</a></td><td><a href="https://github.com/sitaramc">sitaramc</a></td> </tr>
</table>

Hype up your language by improving the current implementation! Give me a shout by [raising an issue](https://github.com/dimroc/etl-language-comparison/issues) to become an owner.

<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/1.0.2/Chart.min.js"></script>
<script>
(function() {
    var options = {
      maintainAspectRatio: true,
      responsive: true
    };

    var results = {
      "ruby": 31.426,
      "ruby-parallel": 7.602,
      "jruby": 12.915,
      "elixir": 10.959,
      "go-substring": 6.040,
      "go-regex": 18.637,
      "python3-pool": 7.331,
      "scala-akka": 8,
      "scala-parallel": 8.5,
      "scala-future": 8.5,
    };

    var results2 = {
      "ruby": 31.426,
      "nim": 2.813,
      "nodejs-cluster": 2.582,
      "php-ascii-single-thread": 10.901,
      "erlang-unsafe-cheats": 2.589,
      "erlang-binary": 4.081,
      "erlang-regex": 5.874,
      "rust-substring": 1.960,
      "rust-regex": 1.382
    };

    function createBarChart(results, canvasId) {
      var data = {
          labels: $.map(results, function(key, element) { return element }),
          datasets: [ { data: $.map(results, function(key, element) { return key }) } ]
      };

      var ctx = document.getElementById(canvasId).getContext("2d");
      var myBarChart = new Chart(ctx).Bar(data, options);
    }

    createBarChart(results, "languageChart");
    createBarChart(results2, "languageChart2");

})();
</script>
