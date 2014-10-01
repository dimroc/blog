---
layout: post
title: A better error handling future for iOS with Swift
tags: swift ios
---

Pattern matching is a fantastic mechanism prevalent in functional programming languages, and [Swift](https://developer.apple.com/swift/) has it! Below, we'll find out how to use it to encapsulate errors in asynchronous code.

<!--more-->

### Swift Enumerations

Define different cases, resulting in new classes scoped to the enum.

{% highlight swift %}
enum GoogleGeocoderResponse {
    case Response(MFLocation)
    case Error(NSError)
}
{% endhighlight %}

Instantiate one of the enumbs depending on the scenario:

{% highlight swift %}
manager.GET(baseUrl(), parameters: parameters, success: { (operation: AFHTTPRequestOperation!, response) in
            let location = GoogleGeocoder.fromResponse(response!)
            callback(GoogleGeocoderResponse.Response(location))
        }) { (operation, error) in
            callback(GoogleGeocoderResponse.Error(error))
        }
{% endhighlight %}

Use the `switch` syntax to perform pattern matching against these enum classes:

{% highlight swift %}
GoogleGeocoder.reverse(location.coordinate, callback: { (response: GoogleGeocoderResponse) in
            switch response {
            case .Error(let error):
                NSLog(error.description)
            case .Response(let location):
                NSLog(location.description)
            }

            self.callback(response)
        })
{% endhighlight %}

One of the benefits to pattern matching is speed. Although one might infer run-time type checking in the `switch`, it is all mostly done at compile time via polymorphism or some other mechanism.

This does not eliminate the need for `NSError`. That approach is still recommended when interacting with UIKit, etc.

Check out [Swiftlytyping's post on Swift error handling](http://swiftlytyping.tumblr.com/post/88210131086/error-handling?utm_campaign=iOS_Dev_Weekly_Issue_150&utm_medium=email&utm_source=iOS%2BDev%2BWeekly)
for a more thorough example with generics.
