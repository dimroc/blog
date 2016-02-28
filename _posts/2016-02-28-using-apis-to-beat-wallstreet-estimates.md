---
layout: post
title: "Using Twitter API to beat the [Wall] Street"
date: "Sun Feb 28 12:23:53 -0500 2016"
tags: business finance
---

I made a killer trade the other day. It was a near guarateed win. And it came from a buy signal on a side project. Let me walk you through it.

I've been farming tweets from cities for years now. They've spun out into experimental projects like [Urban Events](/2015/12/29/search-across-cities/) and
[New Tweet City](/2014/09/24/tweets-as-pixels/). When investigating Twitter media, one thing that became very apparent was that most twitter images actually
belonged to Instagram (IG).

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr"><a href="https://twitter.com/HunterEliteGene">@HunterEliteGene</a> And your favorite level?</p>&mdash; Minibar Austin (@MinibarAustin) <a href="https://twitter.com/MinibarAustin/status/703329275970244611">February 26, 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

This tweet, despite being all text, is actually classified as an `image` in the Twitter API. And this is how it ends up in my application:

{% highlight javascript %}
{
  "id":"703309386622615553",
  "mediaType":"image", // *Important* Marked as Image
  "mediaUrl":"https://scontent.cdninstagram.com/t51.2885-15/s640x640/sh0.08/e35/12724735_1711188682429506_1029304848_n.jpg?ig_cache_key=MTE5MzY3MzU1NjQ3NTU2NjIxMA%3D%3D.2.l", // has instagram url
  "text":"I captured every level of drunk last night 📷🍺 @ 6th Street, Austin, TX https://t.co/ZEspPWrfPR",
  "city":"austin",
  "createdAt":"2016-02-26T20:03:19Z",
  "fullName":"imdotish",
  ...
}
{% endhighlight %}

I knew a lot of the traffic was Instagram but I wasn't sure how much. Good thing I fed all my tweets into Elasticsearch, because it has a data visualization
tool called [Kibana](https://www.elastic.co/products/kibana) that allowed me to chart what percentage of twitter images were actually Instagram links.
Let's start with New York City:

![Twitter NYC Stacked](/public/images/twitterbeatdastreet/TwitterNycImagesStacked.jpg)

Man that's a lot of instagram. Let's check it out as a percentage.

![Twitter NYC Percentage](/public/images/twitterbeatdastreet/TwitterNycMediaSplit.jpg)

Damn bruv. Peaked at 92%! So here's where I give you the disclaimer about which tweets I'm analyzing.
**This is only applicable to geotagged tweets with media.** After all, if it's not geotagged I can't tell if it's coming out of NYC.

Still. Let's take a look at some other cities:

![Twitter Cities Percentage](/public/images/twitterbeatdastreet/TwitterMediaOriginSplitsStacked.jpg)

Shout out to LA for **only having 64% of traffic from Instagram**. So on Twitter's best day, it only has 36% of images on
its own platform.

<img src="/public/images/twitterbeatdastreet/Bruh.jpeg" width="300px" alt="Bruh"/>

## How does Wall Street come in?

I've been getting fleeced by the [TWTR](https://www.google.com/finance?q=TWTR&ei=rxbTVvGnLNaNmAGo2q64Ag) stock for a while now, **even while having this knowledge months ago.**
I knew of this in the Summer of 2015 and did nothing. I was reluctant to use the Twitter API as a buy or sell signal
on the stock. My finance world and my development world were completely distinct. And then this happened.

<img src="/public/images/twitterbeatdastreet/VennEpiphany.jpg" alt="Venn Epiphany" style="max-width: 400px; width: 100%;"/>

It seems obvious now, but I never went into my side projects to discover signals, and looking at the information, the perfect trade
would have been a pair trade shorting TWTR and buying FB (owner of Instagram). But I felt I had missed the boat on TWTR. I actually hadn't,
but I've been burned by bottom fishing before and I wasn't gonna do that again. Not me, no sir.

Then in January, I landed on this [nugget of information about Instagram](http://blog.business.instagram.com/post/128686033016/150909-advertisinglaunch):

> We’re excited to announce that starting this month, advertisers both large and small can run campaigns on Instagram. In addition, ads are now available in more than 30 new countries—including Italy, Spain, Mexico, India and South Korea—and will be launching in markets around the world on Sept. 30.

September 30th 2015. Which means that [FB Q4 Earnings](http://investor.fb.com/releasedetail.cfm?ReleaseID=952040), released in January, will be the first earnings release with IG ad revenue for a full quarter.
My, my are the stars aligning! Couple this with some napkin math on their estimates (**translation**: pure speculation), I was going in for the buy.

## Buying options for leverage right before earnings release

![Facebook Options Confirmation](/public/images/twitterbeatdastreet/FBOptionConfirmation.jpg)

It's not a game anymore. The options had a strike price of $99 and cost me just under $900 (that's all I risked, I know, don't tell me about it).

When the market bell closed on the 28th, [FB](https://www.google.com/finance?q=NASDAQ%3AFB&ei=shbTVuHoJpazmAGNw5HQBQ) stock dipped to $94.50. GG y'all, 🍻, this was fun.

But wait, then earnings were released. It spiked up to as high as $112. I waited until the next morning and I was out of there:

![Facebook Call Options BOOM](/public/images/twitterbeatdastreet/FacebookCallOptionsBOOM.jpg)

Exercising 300 FB shares became $32,748, I got my first margin call and needed to shore up cash. I just dumped the shares
and took home a profit of around $3k.

## All of that for $3K

It was by far the surest I had ever been that a stock would pop on earnings, and I was timid. I needed some Wolf of Wall Street that day
and I didn't have it. But you always say that after a successful trade! It was a great experience and I'm hoping that the stars
will align again, but I've accepted that it might not happen for months, quarters, or even years.

Comment below!
