---
layout: post
title: "video-depth-maps"
date: "Fri Feb 03 15:55:29 -0800 2017"
---

My latest experiment has been to generate video depth maps with the goal of splicing in 3D content.

<!--more-->

It all starts with a stereo video:

<video src="/public/videos/elsegundo-leftright.mp4" controls></video>

and after some math, becomes a left and depth video:

<video src="/public/videos/elsegundo-leftdepth.mp4" controls></video>

## How is this magic possible?

I had a lot of help from the [Zed stereo camera](https://www.stereolabs.com/):

![Zed Product](/public/images/ZED_product_main.jpg)

They are using [Nvidia's CUDA](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=2&cad=rja&uact=8&ved=0ahUKEwj5iNHGlPXRAhUqj1QKHV2ZApwQFggqMAE&url=http%3A%2F%2Fwww.nvidia.com%2Fobject%2Fcuda_home_new.html&usg=AFQjCNFOgRLjdcy04deySQVzAVHfj9Pbiw&sig2=KDG8MuXe2l5WwLJUJNietA&bvm=bv.146094739,d.cGw)
to run GPGPU algorithms against the stereo images to calculate the depth of each pixel.

![Baseline Depth Calculation](/public/images/stereo-geometry.png)
Read more about it [here](http://www.slideshare.net/yuhuang/passive-stereo-vision-with-deep-learning).

## Not Always Perfect

<video src="/public/videos/pier_high_exposure.mp4" controls></video>

As you can see in the video above, distance dramatically affects the depth perception. This is because the distance the camera are from each other, bifocal length, dictates
how much disparity it can calculate in the stereo images. This is why the iPhone 7 Plus can only do portrait mode at 8 feet away.

## Future

This is all extremely relevant in the realm of autonomous vehicles. And that space is on fire. Just look at nVidia's stock the last year:

![Nvidia Stock](/public/images/nvidia_stock.jpg)

