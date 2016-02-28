---
layout: post
title: "Charting Tweets with Kibana 4 and CloudWatch"
date: "Mon Aug 03 07:43:55 -0400 2015"
tags: elasticsearch twitter
---

A lot of people have a lot of data in their Elasticsearch clusters. We have all the geotagged tweets
coming out of cities for the past few months. Extracting any additional insight is a big win,
and this is where [Kibana](https://www.elastic.co/products/kibana) comes in.

![Kibana For Tweets](/public/images/KibanaForTweets.png)

My city tweets are pretty noisy at the moment. I'll check back in when I've sliced out something meaningful.

In the mean-time, I've been tracking the tweet frequency of certain cities via Amazon's [CloudWatch](http://aws.amazon.com/cloudwatch/).
Mostly so [SNS](http://aws.amazon.com/sns/) can send me a text when things go quiet, but it makes for a decent time-series chart:

![CloudWatch For Tweets](/public/images/CloudWatchForTweets.png)

Three takeaways here:

- The day night cycles present an obvious pattern, although it never truly goes quiet.
- NYC and LA have a dramatically higher volume than the other cities, which makes sense given their size.
- Parisians have better things to do than tweet.
