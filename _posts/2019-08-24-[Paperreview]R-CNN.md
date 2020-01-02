---
layout: post
title:  "[Paper review] R-CNN"
date:   2019-08-24 00:01:00 +0900
lang: ko
tags: DeepLearning Paper
---
<hr>

## Rich feature hierarchies for accurate object detection and semantic segmentation ##


## 1) Introduction ##
&nbsp;&nbsp;고전의 **visual recognition task**는 SIFT와 HOG 알고리즘 기반으로 수행했다. 하지만 R-CNN에서는 Convolutional Neural Networks(CNNs)를 통해 여러 태스크에서 극적으로 향상된 object detection을 보여주었다. 이 페이퍼에서는 두 가지의 문제를 찝었다.

1. Deep Network를 통해 object localizing
2. Training a high-capacity model

Detection의 경우는 이미지에서 물체를 가능한 많이 localizing을 해야 한다. 하지만 regression으로는 performance가 낮고 sliding-window detector를 쓸 경우 network에서 unit이 커져 너무 많은 localization이 이루어지고 이는 overfitting을 일으킨다.

따라서 R-CNN에서는 **recognition using regions"이라는 기법으로 위와 같은 CNN localization 문제를 해결하고자 하였다. 간략하게는 다음과 같은 방법을 제시했다.



1. 2000개의 독립된 카테고리가 있을 region proposal을 input image와 함께 CNN을 사용하여 각 proposal(물체가 있을 후보)으로부터 fixed-length feature vector를 얻는다.
2. 각 카테고리에 특화된 specific linear SVM으로 각 영역을 분류한다.

## 2) Object detection with R-CNN ##
&nbsp;&nbsp;R-CNN에서 object detection system은 3개의 module로 이루어져 있다.

![]({{ site.url }}/assets/img/rcnn/rcnn1.png){: width="70%" height="70%" .center}

1. category-independent region proposal를 만드는 module, 이 때 proposal은 detector에 있는 candidate detection을 의미한다.

2. 각 region으로부터 fixed-lenght feature vector을 추출하기 위해 large CNN을 사용한다.

3. class-specific linear SVM을 분류기로 사용한다.

각 module을 어떻게 구성했는 지 살펴보자.

### Module design ###

1. Region proposals<br>
category-independent region proposal은 가능한 이미지 영역이며 이를 찾아내기 위한 다양한 알고리즘이 있지만 R-CNN에서는 selective search를 사용한다. 이는 색상이나 강도, 패턴이 비슷한 인접한 픽셀을 합치는 방식이다.

2. Feature extraction<br>
Region proposals을 통해 찾은 2000 region proprosal으로부터 feature vectore를 추출한다. 이 때 CNN의 input은 고정 사이즈의 image여야 하므로 227 * 227 사이즈로 afffine image warping을 진행한다. (이 때 bounding box가 안으로 들어오도록 필요한 사이즈에 딱 맞게 이미지를 warping한다.)

3. Classifier<br>
class별로 훈련된 SVM을 사용하여 추출된 feature vector에 스코어를 매긴다. 이 때 이미지의 스코어가 매겨진 모든 region은 greedy non-maximum suppression을 적용한다. 이는 Intersection-over-Union(IoU) overlap을 방지한다.

4. Bounding-box regression<br>
localization error을 줄이고 더 정확한 bounding box를 얻기 위해 linear regression을 수행한다.


## 3) Reference ##
[1] paper : https://arxiv.org/abs/1311.2524