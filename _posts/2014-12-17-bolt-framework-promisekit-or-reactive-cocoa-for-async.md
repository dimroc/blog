---
layout: post
title: "Taming Callbacks in iOS: Bolts Framework"
date: "Wed Dec 17 18:53:26 -0500 2014"
tags: ios swift
---

Managing many asynchronous events in iOS can get hairy, but it's a fact of life when working with
remote services. So how do we avoid callback hell?

**Promises**.

Below I'll explain why I chose the [**Bolts Framework (BF)**](https://github.com/BoltsFramework/Bolts-iOS)
over [PromiseKit](https://github.com/mxcl/PromiseKit) and [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa).

<!--more-->

A naive way to handle asynchronous code in Swift is with standard blocks. Let's say we need to change the orientation of a video,
save a thumbnail, and then save the video for a post. We could end up with something like this:

{% highlight swift %}

videoAsset.fixOrientation({ (response: MFVideoAssetResponse!) -> Void in
    if(response.success) {
        let imageData = UIImageJPEGRepresentation(response.thumbnail, 1)
        let imageFile = PFFile(data: imageData, contentType: "image/jpg")

        imageFile.saveInBackgroundWithBlock({ (succeeded: Bool, error: NSError!) -> Void in
            if(succeeded) {
                let videoData = NSData.dataWithContentsOfMappedFile(response.url.path!) as NSData
                let videoFile = PFFile(data: videoData, contentType: "video/quicktime")

                videoFile.saveInBackgroundWithBlock({ (succeeded: Bool, error: NSError!) -> Void in
                    if(succeeded) {
                        post.saveEventually()
                    } else {
                        println("## \(error.description)") // Horrible Error Handler 1!
                    }
                })
            }
        })
    } else {
        println("## FAILED \(response.error.description)") // Horrible Error Handler 2!
    }
})

{% endhighlight %}

Here's the callback pyramid of doom. If you're unfamiliar with this, you can read more about it [here (article is on javascript, but concept still applies)](http://blogs.telerik.com/kendoui/posts/13-03-28/what-is-the-point-of-promises).

We don't want this, and promises are the way to fix it. But which framework should we use?

### [Reactive Cocoa?](https://github.com/ReactiveCocoa/ReactiveCocoa#chaining-dependent-operations)

* Based on the principals of Functional Reactive Programming.
* Required one to wrap everything with **RAC** types.
* Focused on handling streams, not my primary use case.
* When all I need are promises, RAC was too heavy and obtrusive for my tastes.

### [PromiseKit?](http://promisekit.org/)

* Inspired by Bolts.
* Separate library for Objective C and Swift, so it can take advantage of Swift generics.
* Comes with Objective C categories for many built in iOS types, like **NSURLConnection**.
* At the time of this writing, it does not have a pod for the Swift library because of CocoaPod's limitations.
* Not straightforward when switching between Objective C and Swift.

### [Bolts Framework's BFTask?](https://github.com/BoltsFramework/Bolts-iOS) Yes.

* The simplest of the three.
* Works seamlessly across Objective C and Swift.
* Does not support Swift generics which can lead to some verbosity.
* Supported natively by the Parse Framework, which is what I'm using.

I actually started with PromiseKit and just went to Bolts because I found it the simplest of the three. At first, I was all about the Swift support,
but found myself going back and forth between Obj C and Swift, making Bolts a better match. As CocoaPods and Swift matures, I will definitely take
another glance at PromiseKit. Right now, I'm extremely happy with BFTask:


{% highlight swift %}

class func createAsync(videoAsset: MFVideoAsset) -> BFTask! {
    return videoAsset.fixOrientation().continueWithSuccessBlock { (task) -> AnyObject! in
        let response: MFVideoAssetResponse! = task.result as MFVideoAssetResponse;
        println("## Video Orientation Fixed to \(response.url)")

        let imageData = UIImageJPEGRepresentation(response.thumbnail, 1)
        let imageFile = PFFile(data: imageData, contentType: "image/jpg")

        let videoData = NSData.dataWithContentsOfMappedFile(response.url.path!) as NSData
        let videoFile = PFFile(data: videoData, contentType: "video/quicktime")

        let parallelTasks: NSMutableArray = [imageFile.saveInBackground(), videoFile.saveInBackground()]
        return BFTask(forCompletionOfAllTasks:parallelTasks).continueWithSuccessBlock({(task)-> AnyObject! in
            var post = PFObject(className: "Post")
            post["type"] = "video"
            post["image"] = imageFile
            post["video"] = videoFile
            return post.saveEventuallyAsTask()
        })
    }
}

{% endhighlight %}

And better error handling code because in true promise fashion, the caller can receive the error at the end regardless of where it happens:

{% highlight swift %}

PostRepository.createAsync(videoAsset).continueWithBlock(task: BFTask!) -> AnyObject! {
    if(task.success) {
        var post = task.result as PFObject
        println(post)
    } else {
        // Better Error Handler!
        self.presentViewController(
            UIAlertControllerFactory.ok("Error Saving Post", message: task.error.description),
            animated: true,
            completion: nil)
    }
    return nil
}

{% endhighlight %}

## Takeaway

* Smash your iOS pyramids of doom with [Bolts](https://github.com/BoltsFramework/Bolts-iOS) or [PromiseKit](www.promisekit.org).
* Handle to streams of potentially infinite data with ReactiveCocoa.
