---
layout: post
title: "Counting Crowds and Lines"
date: "Sun Nov 19 17:06:05 -0500 2017"
tags: kubernetes machine-learning python aws
---

_This post details the creation of the first version of [countingcompany.com](http://www.countingcompany.com),
check it out!_

In Union Square, NYC, there's the untoppable burger joint name Shake Shack that's
always crowded. A group of us would obsessively check the [Shake Cam](https://www.shakeshack.com/location/madison-square-park/)
around lunch to figure out if that trip was worth it.

<a href="https://www.shakeshack.com/location/madison-square-park" target="_blank">
<img src="/public/images/count/shakecam.jpg" alt="Shake Cam" width="200px"/>
</a>

Rather than do this manually (come on, it's nearly 2018), it would be great if this could be done
for us. Object detection has received a lot of attention in the deep learning space, but it's
ill-suited for scenes with highly congested objects like crowds. In this post, I'll talk about
how I implemented [multi-scale convolutional neural network (CNN)](https://arxiv.org/pdf/1702.02359.pdf)
for crowd and line counting.

<a href="http://www.countingcompany.com" target="_blank">
<img src="/public/images/count/countLineDualShot.jpg" alt="Count Alpha" width="600px"/>
</a>

## Why not object detection

Regional-CNN's (R-CNN) use a sliding window to find an object. High density crowds are ill-suited for
sliding windows due to high occlusion:

<img src="/public/images/count/rcnnfail.jpg" alt="R-CNN" width="300px"/>
<center><small>Failed attempt with off the shelf (no retraining) TensorFlow R-CNN</small></center>

Further exploration in this approach led me to [TensorBox](https://github.com/Russell91/TensorBox),
but it too had issues with high congestion and large crowd counts.

## Density Maps to the rescue

Hence the reason for an approach that uses a density map (akin to a heat map) to better
estimate crowds:
<img src="/public/images/count/ucfOriginal.jpg" alt="UCF Original" width="400px"/>
<img src="/public/images/count/ucfcrowd.jpg" alt="Dense Crowd Ground Truth" width="500px"/>
<center><a href="http://crcv.ucf.edu/data/crowd.php" target="_blank"><small>
Crowd photo from the UCF Dataset
</small></a></center>

Based on [multi-scale convolutional neural network (CNN) for crowd counting](https://arxiv.org/pdf/1702.02359.pdf),
one trains the network to output values from zero to one wherever it thinks there is a head. The sum
of all these pixels then results in the count of the crowd.

Let's look at density maps applied to the shake cam:
<img src="/public/images/count/denseCrowdGroundTruth.jpg" alt="Dense Crowd Ground Truth" width="500px"/>

## How to get the images?

From your neighborhood [Shake Shack Cam](https://www.shakeshack.com/location/madison-square-park) of course.

## How to annotate the data?

The tried and true AWS Mechanical Turk, with a twist: a mouse click annotates a head as shown below:
<img src="/public/images/count/headAnnotator.jpg" alt="Head Annotator" width="400px"/>

I went ahead and modified the [bbox-annotator](https://github.com/kyamagu/bbox-annotator)
to be a single click [head annotator](https://github.com/dimroc/head-annotator).

## How to count the crowd?

As mentioned earlier, one trains the network to output pixel values close to one wherever it thinks there is a head. The sum
of all these pixels then results in the count of the crowd.

<img src="/public/images/count/ucfcrowd.jpg" alt="Dense Crowd Ground Truth" width="500px"/>
<center><small>Just the sum of the pixel values</small></center>

## How to count the line?

Lines aren't merely people in a certain space, they are people standing next to each other
to form a contiguous collection of people. As of now, I simple feed the density map into a
three layer fully connected (FC) network to output a single number, the line count.

Gathering data for that also ended up being a task in AWS Mechanical Turk.

## Making a product out of data science

This is all good fun working on your development box, how do you host it? This
will be a topic for another blog post, but the short story is:

1. Create a docker image with Conda dependencies.
2. Deploy to GCP with kubernetes on Google Container Engine.
3. Periodically run a background job to scrape the shake cam image and run a prediction.

I did the extra credit step of having a Rails application interact with the ML service
via [gRPC](https://grpc.io/), while integration testing with [PyCall](https://github.com/mrkn/pycall.rb).
Not necessary, but I'm very happy with the setup.

## Unexpected Challenges

1. Umbrellas. Not a head but still a person.
2. Shadows. Around noon there can be some strong shadows resembling people.
3. Winter Darkness. It gets much darker much sooner in November and December. Yet the model was trained predominantly with images of people in daylight.

## Check it out

[Counting Company](http://www.countingcompany.com)

<a href="http://www.countingcompany.com" target="_blank">
<img src="/public/images/count/countLineDualShot.jpg" alt="Count Alpha" width="600px"/>
</a>

_Feel free to drop a line below if you have any questions._
