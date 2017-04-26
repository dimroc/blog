---
layout: post
title: "Machine Learning Selfie Segmentation"
date: "Tue Apr 25 22:25:38 -0400 2017"
---

# Machine Learning My Face!

[_Inspired by Automatic Portrait Segmentation for Image Stylization_](http://xiaoyongshen.me/webpage_portrait/index.html)

Selfies are dominating photography to the point where even Narcissus would blush. In my continuing efforts to
ramp up on machine learning and neural networks, I apply a technique called object segmentation to my face
with mixed results but a promising future. This is all heavily inspired by the paper
[Automatic Portrait Segmentation for Image Stylization by Xiaoyong Shen, et al.](http://xiaoyongshen.me/webpage_portrait/index.html)

![Selfie Segmentation And Styling](/public/images/machine-learning-selfie-segmentation/mlPortraitsAllFour.jpg)

This road was longer than I thought it would be. The standard neural networks that people are introduced to,
referred to as fully connected layers (FC), don't suffice with images because there are simply too many features
with a pixel per feature. What to do?

## Convolutional Neural Networks (CNNs)

Neural Networks for images essentially. These connected layers of kernels (or filters) to detect
features that a collection of pixels have, such as edges. For more information, read
these excellent articles:

- [Intuitive Explanation of Convolutional Neural Networks](https://ujjwalkarn.me/2016/08/11/intuitive-explanation-convnets/)
- [Architecture of Convolutional Neural Networks](http://cs231n.github.io/convolutional-networks/)

Xiaoyong Shen, et al. started off by fine-tuning an existing CNN for portraits.
This CNN is known as the [Fully Convolutional Network, or FCN,](https://people.eecs.berkeley.edu/~jonlong/long_shelhamer_fcn.pdf)
and is extremely popular. The result was a Portrait FCN, and that's what the focus of this post is about.

## Portrait FCN

TODO

