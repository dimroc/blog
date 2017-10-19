---
layout: post
title: "Automatic Selfie Segmentation and Style Transfer"
date: "Tue Apr 25 22:25:38 -0400 2017"
tags: python machine-learning
---

[_Inspired by Automatic Portrait Segmentation for Image Stylization_](http://xiaoyongshen.me/webpage_portrait/index.html)
and _[Fast Style Transfer](https://github.com/lengstrom/fast-style-transfer)_.

Selfies are dominating photography, so why not experiment in that space? As I ramp up
on machine learning and neural networks, I apply a technique called object segmentation to my face
with mixed results but a promising future. This is all heavily inspired by the paper
[Automatic Portrait Segmentation for Image Stylization by Xiaoyong Shen, et al.](http://xiaoyongshen.me/webpage_portrait/index.html)

<video src="/public/videos/allFourOutputUdniePhotoshop.mp4" controls="true" type="video/mp4" poster="/public/images/machine-learning-selfie-segmentation/mlPortraitsAllFour.jpg"></video>

Flow:

1. Extract frames from video
2. Generate matte using [portrait segmentation](http://xiaoyongshen.me/webpage_portrait/index.html) on each frame
  - Uses pixel level classification between two categories: foreground and background
3. [Style transfer](https://github.com/lengstrom/fast-style-transfer) on original frame for cartoon effect
4. Cut out foreground by using the matte as a mask on the styled frame
5. Composite new video by placing foreground over original video

This road was longer than I thought it would be. The standard neural networks that people are introduced to,
referred to as fully connected layers (FC), don't suffice with images because it cannot scale up to many pixels.
There are simply too many nodes in the network. Instead, we use the increasingly popular Convolutional Neural Networks
that are particularly good at classifying images. But rather than merely classifying the image as a face,
we want to classify each pixel as either foreground or background.

For this, we use a [Fully Convolutional Network](https://docs.google.com/presentation/d/10XodYojlW-1iurpUsMoAZknQMS36p7lVIfFZ-Z7V_aY/edit#slide=id.g529579d43_1_292)
to perform pixelwise predictions: pixels in, pixels out.

## Convolutional Neural Networks (CNNs)

Neural Networks for images. These are connected layers of kernels (or filters) to detect
features that a collection of pixels have, such as edges. Imagine a kernel being a 5x5 matrix of values used
to detect image properties at a specific section, or receptive field. For more information, read
these excellent articles:

- [Intuitive Explanation of Convolutional Neural Networks](https://ujjwalkarn.me/2016/08/11/intuitive-explanation-convnets/)
- [Architecture of Convolutional Neural Networks](http://cs231n.github.io/convolutional-networks/)

## Fully Convolutional Networks

CNNs were predominantly used to classify images. Is it a dog or cat?
Then the problem of object segmentation came, extracting the pixels that make up the dog or cat.
[Fully Convolutional Network](https://docs.google.com/presentation/d/10XodYojlW-1iurpUsMoAZknQMS36p7lVIfFZ-Z7V_aY/edit#slide=id.g529579d43_1_292)
solves that problem.

[Xiaoyong Shen, et al.](http://xiaoyongshen.me/webpage_portrait/index.html) fine-tuned the
reference FCN implementation specifically for portraits, and a reimplementation of that is
what you see in this post. It is called the Portrait FCN.

### Artifacts

The matting isn't perfect.

<img src="/public/images/machine-learning-selfie-segmentation/matte_140.jpg" alt="Matte Imperfections" style="max-width:200px"/>

As you can see here, the blotch in the top right is obviously not part of the selfie or the foreground, while the black blotch
in the bottom is part of the foreground.
This is because our Portrait FCN isn't doing the best job it could, but there are better solutions out there
already, such as [Portrait FCN+](http://xiaoyongshen.me/webpage_portrait/index.html) that uses a fixed
portrait trimap to assist the model when generating the matte.

I plan to take another approach however. More on that in the next experiment.

## Style Transfer

Once we have the foreground, we use [Logan Engstrom's style transfer](https://github.com/lengstrom/fast-style-transfer) to take the aesthetic from
a painting, like the one shoanw below, and intelligently apply it to a photo, resulting in a cartoon like effect.
A style reminiscent of [Roger Rabbit](https://www.youtube.com/watch?v=gpDaNqSXxp0).

<img src="/public/images/machine-learning-selfie-segmentation/udnie.jpg" alt="Udnie" style="max-width:300px"/>

<a href="https://www.youtube.com/watch?v=xVJwwWQlQ1o" target="_blank">
  <img src="/public/images/machine-learning-selfie-segmentation/fox_udnie.gif" alt="Fox Udnie" style="max-width:300px"/>
</a>

Here's an example of style transfer on an entire video before matting:

<video src="/public/videos/suit1_scaled.mp4" controls="true" type="video/mp4" style="max-width: 200px"
  poster="/public/images/machine-learning-selfie-segmentation/suit1_scaled.jpg">
</video>

## Wrap Up

This gave me fantastic exposure to machine learning on media and the world of Convolutional Neural Networks,
I plan to continue experimenting in this space, and even forked over for an external GPU (eGPU):

[eGPU 2017 Comparison](https://egpu.io/external-gpu-buyers-guide-2017/)

Up until now, I've been using the amazing [Floyd Hub](https://www.floydhub.com), the Heroku for Deep Learning.
Definitely check it out. Even if you have your own hardware, it's great to have some NVidia K80s at your disposal
to speed up experiements.
