---
layout: post
title: "Machine Learning where your long ID comes from"
date: "Sun Apr 09 19:30:56 -0400 2017"
tags: aws machine-learning
---

_This article makes passing reference to the health industry, and as such
all names and numbers have been replaced with bogus information. The concepts
are still relevant and applicable!_

Everyone has an insurance number, and no one knows their actual insurance policy.
Is it Platinum Shield or Platinum Shield Plus NY? Is it the PPO or Silver plan?

A client wanted to know and I was fresh out of my [Machine Learing Class by
Andrew Ng of Stanford and Coursera](https://www.coursera.org/learn/machine-learning).

## Why not just use Regex?

- Regex might not exist for that particular provider
- Cannot learn and improve over time like a machine learning model can.
- Still unmanageable working with tens if not hundreds of providers' regexes
as opposed to one multiclassifying model


## How do we machine learn a string?

*WXY5678* isn't the greatest thing to plot, so how can I find a decision
boundary around it?

I opted to make each digit a dimension, and simply plot that one character in
it's own plane. Then rely on the multidimensional power of machine learning
to intuit who the originator was.

---- INSERT PICTURE OF ONE DIGIT FEATURE FITTING HERE

Imagine 20 images, one per digit, all collaborating to isolate the pattern.
IDs that are purely random will never be isolated, but insurance numbers definitely
have a pattern to them, and this is how we'll learn them.

### Images to place

![Prediction Searcher](/public/images/machine-learning-ids/Parachute_Policy_Predictor.jpg)
![Features Spreadsheet](/public/images/machine-learning-ids/ParachuteInsuranceNumbersExpandedFeatureList.jpg)
![Heat Map](/public/images/machine-learning-ids/PredictionHeatMap.jpg)
![First Digit - Two](/public/images/machine-learning-ids/firstDigitTwoPolicies.png)
![First Digit - Two](/public/images/machine-learning-ids/firstDigitTwoPoliciesLinearRegression.png)
![First Digit - Three](/public/images/machine-learning-ids/firstDigitThreePolicies.png)
![First Digit - Many](/public/images/machine-learning-ids/firstDigitManyPolicies.png)
![Second Digit - Three](/public/images/machine-learning-ids/secondDigitThreePolicies.png)
