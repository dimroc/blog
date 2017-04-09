---
layout: post
title: "Machine Learning NYC Neighborhoods"
date: "Wed Jan 13 10:15:46 -0500 2016"
tags: aws elixir machine-learning
---

Check out the [NYC Neighborhood Predictor](http://machinelearninghoods.dimroc.com)!
<img src="/public/images/FlyingToCoachellaPrediction.jpg" alt="Prediction Matrix" width="400px"/>

It uses [AWS Machine Learning](https://aws.amazon.com/machine-learning/) to predict
which neighborhood a string of text originates from.

## Overview

- Uses a simple [Elixir and Phoenix](http://www.phoenixframework.org/) Web app to expose an
  AWS Machine Learning [Real-Time Endpoint](http://docs.aws.amazon.com/machine-learning/latest/dg/requesting-real-time-predictions.html).
- [Source Code](https://github.com/dimroc/machine_learning_hoods)

## Machine Learning What?

- Using a dataset of ~1G of geo-tagged tweets, we create a CSV with two columns: text and neighborhood.
- After training and evaluating a machine learning (ML) model with this data, we expose the real-time endpoint via this elixir application.

## Takeaways

- Molding the training data to create a better model is the real challenge here.
- Does my data even have statistical correlations or is it just noise?
- Iterate, iterate, and iterate again on the model and evaluation data is what needs to be done here.


### Input Schema

```json
{
  "version": "1.0",
    "targetAttributeName": "Neighborhood",
    "dataFormat": "CSV",
    "dataFileContainsHeader": true,
    "attributes": [
    {
      "attributeName": "Text",
      "attributeType": "TEXT"
    },
    {
      "attributeName": "Neighborhood",
      "attributeType": "CATEGORICAL"
    }
    ],
    "excludedAttributeNames": []
}
```

![Prediction Matrix](/public/images/PredictionMatrix.jpg)
![Neighborhood Categories](/public/images/NeighborhoodCategories.jpg)


