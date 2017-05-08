---
layout: post
title: "Selfie Segmentation and Style Transfer"
date: "Tue Apr 25 22:25:38 -0400 2017"
tags: python machine-learning
---

[_Inspired by Automatic Portrait Segmentation for Image Stylization_](http://xiaoyongshen.me/webpage_portrait/index.html)
and _[Fast Style Transfer](https://github.com/lengstrom/fast-style-transfer)_.

Selfies are dominating photography, so why not experiment in that space? As I ramp up
on machine learning and neural networks, I apply a technique called object segmentation to my face
with mixed results but a promising future. This is all heavily inspired by the paper
[Automatic Portrait Segmentation for Image Stylization by Xiaoyong Shen, et al.](http://xiaoyongshen.me/webpage_portrait/index.html)

<video src="/public/videos/allFourOutputUdnie.mp4" controls="true" type="video/mp4" poster="/public/images/machine-learning-selfie-segmentation/mlPortraitsAllFour.jpg"></video>

Flow:

1. Extract frames from video
2. Generate matte using [Portrait Segmentation](http://xiaoyongshen.me/webpage_portrait/index.html) on each frame
3. [Style transfer](https://github.com/lengstrom/fast-style-transfer) on original frame
4. Generate foreground by using the matte as a mask on the style frame
5. Composite new video by placing foreground over original video

This road was longer than I thought it would be. The standard neural networks that people are introduced to,
referred to as fully connected layers (FC), don't suffice with images because it cannot scale up to many pixels,
there are simply too many nodes in the network.

## Convolutional Neural Networks (CNNs)

Then came CNNs: Neural Networks for images. These are connected layers of kernels (or filters) to detect
features that a collection of pixels have, such as edges. Imagine a kernel being a 5x5 matrix of values used
to detect image properties at a specific section, or receptive field. For more information, read
these excellent articles:

- [Intuitive Explanation of Convolutional Neural Networks](https://ujjwalkarn.me/2016/08/11/intuitive-explanation-convnets/)
- [Architecture of Convolutional Neural Networks](http://cs231n.github.io/convolutional-networks/)

Xiaoyong Shen, et al. started off by fine-tuning a existing CNN for portraits.
This CNN is known as the [Fully Convolutional Network, or FCN,](https://people.eecs.berkeley.edu/~jonlong/long_shelhamer_fcn.pdf)
and is extremely popular. Shen's first result is known as the Portrait FCN,
a FCN with certain layers retrained against portraits, and that's what's been used for this experiment.

## Style Transfer

Once we have the foreground, we use [Logan Engstrom's style transfer](https://github.com/lengstrom/fast-style-transfer) to give it a cartoon like effect, and then
place it back in the original video. A style reminiscent of [Roger Rabbit](https://www.youtube.com/watch?v=gpDaNqSXxp0).

Here's an example of style transfer on an entire video before matting:

<video src="/public/videos/suit1_scaled.mp4" controls="true" type="video/mp4" poster="/public/images/machine-learning-selfie-segmentation/suit1_scaled.jpg"></video>

## Wrap Up

This gave me fantastic exposure to machine learning on media and the world of Convolutional Neural Networks,
I plan to continue experimenting in this space, and even forked over for some real hardware to boot:

![Fragbox](/public/images/FragBox3-Dark.png)

_[Fragbox](https://www.falcon-nw.com/desktops/fragbox/design)_


Up until now, I've been using the amazing [Floyd Hub](www.floydhub.com), the Heroku for Deep Learning.
Definitely check it out. Even if you have your own hardware, it's great to have some NVidia K80s at your disposal
to speed up experiements.
