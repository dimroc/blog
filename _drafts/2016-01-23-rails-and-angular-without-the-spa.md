---
layout: post
title: "Rails and Angular without the SPA"
date: "Sat Jan 23 19:15:02 -0500 2016"
tags: ruby rails angular
---

A full on angular single page application (SPA) is overkill 90% of the time.
This article will illustrate a simple way to get the rich user experience from Angular and
javascript while still relying on Rails for the bulk of the application.

This approach is based on the premise that only a handful of pages actually
need javascript functionality. For example, you might have a master-detail page,
and want AJAX requests to fill the detail content.

Our approach advocates the following philosophy:

> Javascript and angular functionality exists to enrich the user experience of existing server-side rendered pages.

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

This will callback will bootstrap angular on every turbolinks page navigation.

### Angular Directives and Controllers

With the bootstrap step done, we can now add javascript functionality to our pages.

There are two main ways to do this:

1. Angular Controllers
  - Useful when adding page specific functionality

2. Angular Directives
  - Useful when creating reusable widgets, such as a HTML 5 video player, that can be reused across pages.


## Angular Controllers

TODO

## Angular Directives

TODO

### Relying on Server Side Rendering

If using remote forms:

{% highlight javascript %}
angular.module('ltApp').
  controller('AccessTokenCtrl', ['$scope', '$element', function($scope, $element) {
    $element.on("ajax:success", function(e, data, status) {
      $element.find("#access_token").html(data.html);
    });
}]);
{% endhighlight %}

