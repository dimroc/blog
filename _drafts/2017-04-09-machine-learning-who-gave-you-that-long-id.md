---
layout: post
title: "Machine Learning where your long ID comes from"
date: "Sun Apr 09 19:30:56 -0400 2017"
tags: aws machine-learning
---

_This article talks about a solution that involved sensitive information.
As a result, all the names and numbers have been replaced with bogus
information.  The concepts are still relevant and applicable!_

Everyone has an insurance number, and no one knows their actual insurance policy.
Is it car or life insurance? Is it Platinum Shield Standard or Platinum Shield
Plus NY? Bronze or Silver plan?

A client wanted to know and I was fresh out of my [Machine Learing Class by
Andrew Ng of Stanford and Coursera](https://www.coursera.org/learn/machine-learning).

## Why not just use Regex?

- Regex might not exist for that particular provider
- Cannot learn and improve over time like a machine learning model can.
- Still unmanageable working with tens if not hundreds of providers
as opposed to one multiclassifying model


## How do we machine learn a string?

**WXY5678** isn't the greatest thing to plot, so how do you draw a decision
boundary around it?

I opted to make each digit a dimension, and simply plot that one character in
it's own plane. Then rely on the multidimensional power of machine learning
to intuit who the originator was.

Therefore **WXY5678** becomes an array of seven features, a seven dimensional vector:

```
[W,X,Y,5,6,7,8]
```

We can't plot **W** though can we? Yes we can, by translating [0-9A-Z] to be
a number from 0-35, essentially using our numbers as base 36.

```
[32,33,34,5,6,7,8]
```

## Starting with the first digit: one dimension

![First Digit - Two](/public/images/machine-learning-ids/firstDigitTwoPolicies.png)

Here we have plotted the first digit of two policies for one thousand numbers,
and the patterns are obvious to the human eye. A simple linear regression
would yield the following split:

![First Digit - Two](/public/images/machine-learning-ids/firstDigitTwoPoliciesLinearRegression.png)

We can now confidently (>90%) predict the policy just based on the first digit.
We take this concept and extrapolate it in two directions:

1. Add more policies, and therefore more classes
2. Add more digits, and therefore more dimensions

## More policies, more classes

![First Digit - Three](/public/images/machine-learning-ids/firstDigitThreePolicies.png)

The yellow bar is actually a third policy, and notice that it's hardcoded to
once value, resulting in a point cloud so thick, it looks like a line.
Again, we can easily intuit this policy from the others with high confidence,
with just the first digit.

Will our luck continue as we add more policies?

![First Digit - Many](/public/images/machine-learning-ids/firstDigitManyPolicies.png)

No.

Above, you'll see a plot of just eight policies among the hundred we're trying
to classify. You can see that patterns start falling apart.

But don't lose hope. We have many more digits to go, and that'll help us draw
confident decision boundaries around certain policies. It'll just be in
20D. Do they have glasses for that?

## More digits, more dimensions

Imagine 20 features, one per digit, all collaborating to isolate the pattern.

--- INSERT 3d IMAGE

Visualizing three dimensions can get trippy, twenty plus, unrealistic.
As such, we place our trust in math that the combination of these dimensions
will yield confident results.

Visualizing such high dimensions involve [Dimensionality Reduction](https://en.wikipedia.org/wiki/Dimensionality_reduction)
and is not something I did for this project.

## Other Features

![Features Spreadsheet](/public/images/machine-learning-ids/ParachuteInsuranceNumbersExpandedFeatureList.jpg)

Just for good measure, I added the additional features of number length and base 10.
Policy numbers vary in length based on who issued them, so just another
dimension for us to draw decision boundaries for. The base 36 number as
base 10 was an experimented that yielded little return.


## Final Outcome

![Heat Map](/public/images/machine-learning-ids/PredictionHeatMap.jpg)
*Amazon Machine Learning Model Evaluation*

Above, you can see ten of the hundred policies plotted in accuracy matrix.
The blue boxes descending diagonally indicate correct predictions and show
this has been pretty successful. The F1 Score for the first row is 92% while
the accuracy (not shown) is 99.5%. Not bad!

The two red boxes above might give cause for concern but the mistaken predictions
all belonged to the correct company, just a different plan.
The service is tasked with showing the top three predictions, so the correct
prediction will appear second rather than first, something the service can
live with.

### Policy Predictor

![Prediction Searcher](/public/images/machine-learning-ids/Parachute_Policy_Predictor.jpg)

And there you have it, machine learned predictions at your fingertips.
