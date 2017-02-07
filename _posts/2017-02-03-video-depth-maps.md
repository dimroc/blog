---
layout: post
title: "Video Depth Maps, Zed and NVDA"
date: "Fri Feb 03 15:55:29 -0800 2017"
tags: finance webgl
---

My latest experiment has been to generate video depth maps with the goal of splicing in 3D content.

<!--more-->

It all starts with a stereo video:

<video src="/public/videos/elsegundo-leftright.mp4" controls></video>

and after some math, becomes a __left and depth video__:

<video src="/public/videos/elsegundo-leftdepth.mp4" controls></video>

The whiteness of each pixel in the video to the right dictates how far away that object is from the camera. A completely white pixel is 2 feet away and a gray pixel could be say 10 feet away.

## How is this magic possible?

I had a lot of help from the [Zed stereo camera](https://www.stereolabs.com/):

![Zed Product](/public/images/ZED_product_main.jpg)

They are using [Nvidia's CUDA](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=2&cad=rja&uact=8&ved=0ahUKEwj5iNHGlPXRAhUqj1QKHV2ZApwQFggqMAE&url=http%3A%2F%2Fwww.nvidia.com%2Fobject%2Fcuda_home_new.html&usg=AFQjCNFOgRLjdcy04deySQVzAVHfj9Pbiw&sig2=KDG8MuXe2l5WwLJUJNietA&bvm=bv.146094739,d.cGw)
to run GPGPU algorithms against the stereo images to calculate the depth of each pixel.

## Not Always Perfect

<video src="/public/videos/pier_high_exposure.mp4" controls></video>

As you can see in the video above, distance dramatically affects depth perception. The horizon is obviously __not__ the closest thing to the camera.

The farther away the two lenses are from each other, the farther away it can perceive depth. Inversely, the narrower
the baseline, the closer it can perceive depth. This distance is called the baseline.
The iPhone 7 Plus can only do portrait mode eight feet away because of the baseline length between its dual cameras on such a small form factor.

![Baseline Depth Calculation](/public/images/stereo_depth.jpg)
Read more about it generating depth maps from stereo images [here at OpenCV](http://docs.opencv.org/3.2.0/dd/d53/tutorial_py_depthmap.html).

## Future

This is all extremely relevant in the realm of autonomous vehicles. And that space is on fire. Want to know how far a rock is from a car? Generate a depth map in real time.
And that's exactly what NVidia's graphics cards allow you to do. The market seems excited by it, take a look at nVidia's stock price the last year:

![Nvidia Stock](/public/images/nvidia_stock.jpg)

