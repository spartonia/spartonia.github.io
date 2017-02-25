---
layout: post
title: Dynamic Human identity Recognition Using Kinect Device
categories: [vision]
tags: [google earth, kinect, c++, face recognition, face detection]
comments: true
youtubeId: LtXUzOpKmTM
---
#### A demo of a research project I did at Umea university in Sweden. 

Face recognition has been getting pretty good at full frontal faces and 20 degrees off, but as soon as you go towards profile, they've been prolems. To address this problem, we proposed a method by fusing face recognition and skeletal tracking approaches using a Kinec device. A wall-mounted Kinect is used for capturing RGB and depth (skeletal) data. We first perform a face recognition on RGB images. Once the face is recognized, we assign the corresponding id to the skeleton. In this way, the identity of the tracked skeleton is known- and so the person's- as long as the person is in kinect's view (> 160 degrees). Details and related publication [here](http://ieeexplore.ieee.org/document/6726248/).

{% include youtubePlayer.html id=page.youtubeId %}
