---
layout: post
title: "ETL Language Showdown Part 3"
date: "Aug 31 10:52:19 -0400 2015"
---


#### Rules of Reference Implementation

1. Stream input from files.
2. Use Regular Expressions to check for the presence of `knicks`.
3. Have multiple mappers, but one reducer.
4. Each individual worker holds its results in a hash and sends that final hash back for reduction.

Here are the previous implementations described in [part 2 of the ETL showdown](/2015/05/07/etl-language-showdown-pt2/):

<div height="800px">
  <canvas id="languageChart"></canvas>
</div>

## Shortcuts lead to apple to orange comparisons

It's becoming increasingly difficult to keep the implementations consistent across languages.
Below, you will find underneath each languages how the implementation strays from the original
ruby implementation. It all started when I noticed how much slower Golang's regular expression checks were to substring comparisons.

#### Examples of shortcuts

1. PHP takes a shortcut by using `stripos` that only works on ASCII, not Unicode.
2. Regex comparison as opposed to substring checks.
3. Loading file contents into memory as opposed to streaming.
4. Having the master or supervisor keep track of the results so there is effectively no reduction step.

Instead, use this repo as a reference for writing idiomatic ETL solutions in the language of your choice.

### New School: Language Additions

<div height="800px">
  <canvas id="languageChart2"></canvas>
</div>

### Erlang

1. Leverages [Binary Pattern Matching](http://www.erlang.org/doc/efficiency_guide/binaryhandling.html).
2. Holds all data in memory as opposed to streaming input line by line.

### Nim

1. Implementation seems identical to the ruby version (streaming input with regex but using multiple cores) and it is blazing fast. The first I've heard of [Nim](http://nim-lang.org/).

### Rust

1. Regex solution is identical to reference implementation.
2. Substring solution only deals with ASCII.
3. Man is it fast. Would love someone to take another look at the Rust implementation to see how it might be taking shortcuts against the reference implementation.

### PHP

1. Only doing substring checks against ASCII.
2. Single threaded.
3. Included just to grow the library of implementations. Improvements and pull requests are welcome.

### Node JS

1. Uses cluster to get around the Global Interpreter Lock (GIL).
2. The workers communicate a match to the cluster master, effectively skipping the reduction step.


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