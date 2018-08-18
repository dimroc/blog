---
layout: post
title: "Moving AI from the Cloud to the Edge with Crowd Count and Apple's Core ML"
date: "Sun Aug 12 09:12:27 -0400 2018"
tags: machine-learning ios swift coreml python
---

![iOS Crowd Counting in action](/public/images/count/CrowdCountiOS.gif)

After successfully counting crowds in python, [which you can read about
here](/2017/11/19/counting-crowds-and-lines/), I took it upon myself to run the very same models
on the iPhone with Core ML (ML: Machine Learning).
Core ML and MLKit open up a new domain for computing that's being branded as Edge AI, where neural networks run on a local device (edge) as opposed to the cloud.
The [source code for this experiment is available here](https://github.com/dimroc/count/tree/master/ios).

In this article, we'll go over seven facets of migrating from Cloud AI to Edge AI:

1. Building the Crowd Classifier with Create ML
2. Different MLModels for different Crowd Densities: singles, tens, hundreds+
3. Python coremltools to convert Keras to Core ML
4. Why an Xcode Playground for Core ML became a macOS App (Playgrounds are too brittle)
5. Swift Architecture: RxSwift and MVVM
6. Performance: Better than expected
7. Promising Future for Edge AI

<hr/>

## 1. Building the Crowd Classifier with Create ML

Apple's new [Create ML tool](https://developer.apple.com/documentation/createml)
allows you to build a classifier by simply dragging in a folder.
This was used to classify images as either singles, tens, or hundreds, as a first step
in an ML pipeline. This classification then dictates which counting model to use,
described below.

<img src="/public/images/count/CrowdClassifier.png" alt="Crowd Classifier" width="600px"/>

## 2. Different MLModels for different crowd densities: singles, tens, hundreds+

These crowd counting models (from the previous post) are heavily skewed to particular densities. For example,
the model used to count three people, will have a hard time with one thousand people.
They are based on the paper [Multi-Scale Convolutional Neural Networks for Crowd Counting](https://arxiv.org/pdf/1702.02359.pdf).
To mitigate this, we have three models:

- singles: Uses Apple's built in face detection ([VNDetectFaceRectanglesRequest](https://developer.apple.com/documentation/vision/vndetectfacerectanglesrequest)). You can beat it by showing the back of your head!
- tens: Uses the model overtrained for the [ShakeCam](http://count.dimroc.com/) in the previous crowd counting post, which has between zero to eighty people. The tens category desparately needs a newly trained model.
- hundreds: Uses a model built with the [UCF Crowd dataset](http://crcv.ucf.edu/data/crowd.php).

This is the final step in a two step ML pipeline:

```
Crowd Classification -> Crowd Prediction
```

These models, however, are still in the python Keras format. Let's convert them to Core ML.

## 3. Python coremltools to convert Keras to Core ML

Apple's [coremltools](https://github.com/apple/coremltools) allow one to convert existing
Keras models to Core ML, and that's exactly what was done for the crowd prediction model.

The top variable sized layer had to be replaced with a fixed size layer to better work
with Core ML. Variable sized input has come to Core ML, but support and documentation still has a way to go.

```python
model = load_model(path)

# Create a new input layer to replace the (None,None,3) input layer
input_layer = InputLayer(input_shape=(675, 900, 3), name="input_1")

# Save
intermediary_path = "tmp/reshaped_model.h5"
model.layers[0] = input_layer
model.save(intermediary_path)

# Convert
coreml_model = coremltools.converters.Keras.convert(
    intermediary_path,
    input_names=['input_1'],
    image_input_names=['input_1'],
    output_names=['density_map'])

coreml_model.save("CrowdPredictor.mlmodel")
```

[Source](https://github.com/dimroc/count/blob/master/ml/crowdcount/management/commands/convert_to_coreml.py)

Now that we have our Core ML pipeline, a crowd classifier feeding into a crowd predictor, let's
see it in action.

## 4. Why an Xcode Playground for Core ML became a macOS App (Playgrounds are too brittle)

I was under the impression that the fastest way to a functional prototype was
an Xcode Playground. Inspired by Create ML, the Xcode playground promised to be
a quick and easy prototype for the iOS application.

When it works, it's great.

<img src="/public/images/count/CountPlayground.jpg" alt="Crowd Playground" width="600px"/>

A significant downside to playgrounds however, is its inability to be an Xcode target,
meaning it is unable to have linker and build dependencies. As you use
CocoaPods or Cartography for third party library management, this will break you, and you'll
end up with spurious errors like the follow:

```
error: Couldn't lookup symbols:
  type metadata for CrowdCountApiMac.FriendlyClassification
  ...
  __swift_FORCE_LOAD_$_swiftCoreMedia
  __swift_FORCE_LOAD_$_swiftCoreAudio
  CrowdCountApiMac.FriendlyPredictor.DensityMapWidth.unsafeMutableAddressor : Swift.Int
  ...
```

The good news is that porting over to a macOS application is relativately straighforward, and will fix all of this.
A macOS app is an Xcode target and can therefore link frameworks and binaries, allowing reliable
compilation and linking.

## 5. Swift Architecture: RxSwift and MVVM

[RxSwift](https://github.com/ReactiveX/RxSwift/) drove out an MVVM (Model-View-ViewModel) architecture,
where Apple's ViewControllers are the View. So really M(VC)VM.
But these ViewController's are lean, and merely drive the View or glue the VM.

<img src="/public/images/count/MVCVM.jpg" alt="M(VC)VM" width="600px"/>

There are [many](https://medium.com/@daltonclaybrook/rxswift-mvvm-a-little-at-a-time-81ac17dcf285)
[articles](https://medium.com/@navdeepsingh_2336/creating-an-ios-app-with-mvvm-and-rxswift-in-minutes-b8800633d2e8)
[on this topic](https://medium.com/@dkhuong291/rxswift-with-mvvm-e4af71413298). Feel free to google for more information.

## 6. Performance: Better than expected

While extracting frames from the camera in real time, classification took ~100ms and crowd counting taking ~3 seconds.
Good to see that iPhone X GPU being put to good use.

Word of advice: do not do your own image and MLMultiArray manipulation.
Use Apple's Vision API, such as [VNImageRequestHandler](https://developer.apple.com/documentation/vision/vnimagerequesthandler),
that makes better use of hardware.

## 7. Promising Future for Edge AI

The ability to run sophisticated neural networks, in this case
a multi-column [CNN](https://adeshpande3.github.io/A-Beginner%27s-Guide-To-Understanding-Convolutional-Neural-Networks/),
on your iPhone rather than the cloud, and in real time, is a watershed moment.

Sure, this already existed in applications like Prisma and Google translate, but the ease of development
will present many more opportunities.

With millions of iPhones in use, it'll be easier than ever to crowd source data from
your willing users to improve your model, and create a virtuous cycle that will improve the product:

```
usage -> more data -> improved usage -> more data -> improved usage -> ...
```

This proof of concept shows that it's doable, and that, in and of itself, is a milestone.
Add in the performance and the iPhone's ubiquity, and we have a promising future.
