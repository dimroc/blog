---
layout: post
title: "Rails and Angular without the SPA"
date: "Sat Jan 23 19:15:02 -0500 2016"
tags: ruby rails angular
---

A full on Angular single page application (SPA) is overkill 90% of the time.
This article will illustrate a simple way to get the rich user experience from
javascript while still relying on Rails server rendering for the bulk of the application.

Our approach advocates the following philosophy:

> Javascript and angular functionality exists to enrich the user experience of existing server-side rendered pages.

This is based on the premise that only a handful of pages actually
need javascript functionality. Most of your pages are happily being rendered server side,
but there's that one page that needs sophisticated user experience (UX). For example, you might have
a master-detail page, and want AJAX requests to fill the detail without navigating to a new page.

## Setting up Angular with Rails

I recommend using [bower-rails](https://github.com/rharriso/bower-rails) to manage client side dependencies like Angular.
You will need to install bower, please go to [bower-rails](https://github.com/rharriso/bower-rails) for more details if unfamiliar.

#### `Bowerfile`

{% highlight ruby %}
# A sample Bowerfile
# Check out https://github.com/42dev/bower-rails#ruby-dsl-configuration for more options

asset 'angular', '~> 1.4.8'
{% endhighlight %}

Pull down the client side dependencies

{% highlight bash %}
rake bower:install bower:clean
{% endhighlight %}

#### Then add .js files to `application.js`

{% highlight javascript %}
//= require angular/angular
{% endhighlight %}

Setup is done.

### Working With Turbolinks

If you're using Turbolinks (you should), you'll need to bootstrap
the angular application on every Turbolinks page load.

I recommend creating an `initializer.js` file with the following:

{% highlight javascript %}
// On turbolinks load:
$(document).on('ready page:load', function() {
  angular.bootstrap('body', ['ltApp'])
});
{% endhighlight %}

This callback will bootstrap angular on every turbolinks page navigation.

## Angular Directives and Controllers

With all our setup done, we can now add Angular functionality to our pages.

There are two main ways to do this:

1. Angular Controllers
  - Useful when adding page specific functionality

2. Angular Directives
  - Useful when creating reusable widgets, such as a HTML 5 video player, that can be reused across pages.


## Angular Controllers

Angular let's you tag DOM elements with `ng-controller` to associate javascript functionality with
a particular page. The key word below being `UserCountsChartCtrl`:

#### `users/index.haml.html`
{% highlight haml %}
.graphs{"ng-controller" => "UserCountsChartCtrl"}
  .hidden.graph-data{data: { 'graph-data' => @weekly_data.to_json} }
  .row
    .col-md-12
      %canvas.chart.chart-bar{data: "barGraph.data", labels: "barGraph.labels", series: "barGraph.series", options: "barGraph.options"}
{% endhighlight %}


#### `javacsripts/ng/controllers/UserCountsChartCtrl.js`
{% highlight javascript %}
angular.module('ltApp').
  controller('UserCountsChartCtrl', ['$scope', '$element', function($scope, $element) {
    var labels = $element.find('.graph-data').data("graph-data").dates;
    var data = $element.find('.graph-data').data("graph-data").user_counts;

    // bar graph
    $scope.barGraph = {
      labels: labels,
      series: ["User Counts"],
      data: [data],
      options: { responsive: true, maintainAspectRatio: false }
    };
}]);
{% endhighlight %}

Here I am drawing bar charts on the index page using [Charts.js](http://www.chartjs.org/) with an Angular Controller.

### Passing data from server side to client side
One often needs to get information to the javascript client. In a typical SPA, this
is done with a subsequent AJAX call. Here we save that AJAX call and just render the information
in `data` HTML attributes. Notice the lines below:

{% highlight haml %}
.graphs{"ng-controller" => "UserCountsChartCtrl"}
  .hidden.graph-data{data: { 'graph-data' => @weekly_data.to_json} }
  ....
{% endhighlight %}

#### `javacsripts/ng/controllers/UserCountsChartCtrl.js`
{% highlight javascript %}
angular.module('ltApp').
  controller('UserCountsChartCtrl', ['$scope', '$element', function($scope, $element) {
    var labels = $element.find('.graph-data').data("graph-data").dates; // pull data
    var data = $element.find('.graph-data').data("graph-data").user_counts;
    ...
{% endhighlight %}

This allows you to leverage Rails (controllers, presenters, etc) to present the data needed by the javascript client side.

## Angular Directives

[Angular Directives](https://docs.angularjs.org/guide/directive) allow you to create
javascript functionality that is cross cutting. The example below is ensures that
a form can only be submitted once, to stop those pesky double click submissions.

{% highlight javascript %}
angular.module('ltApp').
    directive('ltSingleSubmit', function () {
      return function (scope, element, attrs) {
        element.bind("submit", function (event) {
          element.find("input[type='submit']").prop('disabled', true);
        });
      };
    });
{% endhighlight %}

This can now be easily used in any form:

{% highlight haml %}
  = simple_form_for @build, url: build_path(@build), html: { "lt-single-submit" => '' } do |f|
    = f.input :name
    = f.submit 'Save'

{% endhighlight %}

### Relying on Server Side Rendering

When making hybrid applications like this, it's easy to get server side
and client side rendering jumbled together. I advocate always using server side rendering.
Sure it's less flashy, but it's more productive and consolidates your validations into one place.

I've done this many ways. We'll start with the simplest: Angular $http GETs.

### AJAX with Angular $http

{% highlight javascript %}
angular.module('ltApp').
  controller('MarketingShowCtrl', ['$scope', '$element', '$http', 'turbosafe', function($scope, $element, $http, turbosafe) {
    var path = $element.find(".cta").data("path"); // Get the path for the GET form the data attribute.

    // Angular Scope has method fetchCollection used by ng-click.
    $scope.fetchCollection = function(collectionName, direction) {
      // Start Turbolinks progress bar even though we're using Angular!
      turbosafe.start();

      $http.get(path, {params: {id: collectionName, direction: direction}}).
        then(function(response) {
          var newDom = $(response.data);
          // Replace DOM element with Server Side rendered HTML.
          $element.find("#top").replaceWith(newDom);
          turbosafe.stop();
        }, function(response) {
          console.warn("Unable to retrieve collection", response);
          turbosafe.stop();
      });
    };
{% endhighlight %}

{% highlight haml %}
= link_to "Next Collection", "#", "ng-click" => 'fetchCollection("$50", "Next")'
{% endhighlight %}

### AJAX with Remote Forms

{% highlight javascript %}
angular.module('ltApp').
  controller('AccessTokenCtrl', ['$scope', '$element', function($scope, $element) {
    // Handle remote forms success.
    $element.on("ajax:success", function(e, data, status) {
      $element.find("#access_token").html(data.html);
    });
}]);
{% endhighlight %}

{% highlight haml %}
.panel-body
  .row
    .col-md-12{ "ng-controller" => "AccessTokenCtrl"}
      %h3.col-header
      #access_token= @user.live_access_token
      - # Notice the remote: true to show Rails remote forms.
      = simple_form_for(@user, url: generate_access_token_account_path(mode: :live), remote: true) do |f|
        = f.button :button do
          %span.glyphicon.glyphicon-refresh
{% endhighlight %}

## Wrap up

You're right, I got carried away at the end. But now you have an exhaustive set of examples
that show how a sprinkle of Angular can breathe rich interactivity into your Rails rendered pages.

Enjoy!
