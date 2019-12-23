---
layout: post
title:  "[Paper review] SPADE"
date:   2019-09-18 00:02:00 +0900
lang: ko
tags: DeepLearning
---

## Semantic Image Synthesis with SPADE ##
<hr>

&nbsp;&nbsp;*오랜만에 GAN 관련 논문을 읽고 SPADE를 너무 감명깊게 본 나머지 리뷰를 꼭 남겨야 할 것 같아서 진행한다.*

## 1) Introduction ##
&nbsp;&nbsp;이 논문의 task는 semantic segmentation mask에서 매우 사실적인 사진 이미지로 translation하는 것이다. 이전의 연구에서는 conv, normalization, nonlinearity layer으로 network를 만들었지만 이 때 <u>normalization layer에서는 input semantic mask의 정보를 잃는다는 단점</u>이 있다.

따라서 이를 해결하기 위해 spatially adaptive를 이용해 activations를 조절한 conditional normalization layer인 **Spatially-adaptive normalization**을 제안했다. 이는 transformation을 학습하고 network를 통해 semantic 정보를 효율적으로 전파시킨다.


## 2) Related Work ##

### 2-1) Deep generative models ###
* SPADE는 GAN의 구조를 갖지만 conditional image synthesis task를 목표로 한다.

### 2-2) Conditional image synthesis ###
* 다양한 형태의 input data가 있지만 여기서는 **segmentation mask와 image 쌍**을 사용하여 spatially-adaptive normalization을 통해 segmentation mask에서 photorealistic image를 만든다.

### 2-3) Unconditional normalization layers ###


![]({{ site.url }}/assets/img/rcnn/fasterrcnn1.png){: width="70%" height="70%" .center}

Faster R-CNN은 2개의 module로 이루어져 있다.
1. region을 계산하는 **deep fully convolutional network**
2. 계산된 region을 이용한 Fast R-CNN detector

그리고 이 과정은 모두 위의 구조처럼 하나의 unify된 network로 구성되어 있다. 이 때 위에서 설명했던 **RPN** module을 사용하므로 기존의 Fast R-CNN과는 구별된다.

### 2-1) Region Proposal Networks ###
&nbsp;&nbsp;**Region Proposal Networks(RPN)** 은 다양한 크기의 이미지를 넣어 직사각형의 object proposal들을 출력하는 데 각각은 objectness score을 나타낸다. 이 과정은 FCN으로 구성되는 데 기존의 Fast R-CNN object detection network와 RPN의 <u>computation을 공유하기 위함</u>이다.


&nbsp;&nbsp;Region proposal을 만들기 위해 연산을 공유하는 마지막 conv layer에서 나온 **conv feature map**에 small network를 **슬라이딩**시킨다. 이 small network는 conv feature map을 input으로 하는 **n ✕ n size의 spatial window**로 이루어져 있다. sliding window는 intermediate layer를 통해 **lower-dimensional feature**로 매핑된다. 이 논문에서 실험한 intermediate layer는 두 가지이다.

1. 256-d for Zeiler and Fergus model(ZF), which has 5 shareable conv + ReLU
2. 512-d for VGG-16, which has 13 shareable conv + ReLU

이를 통해 나온 feature들은 두 개의 **Fully connected layers**를 통과한다.
1. Box-regression layer(reg)
2. Box-classification layer(cls)

![]({{ site.url }}/assets/img/rcnn/fasterrcnn2.png){: width="70%" height="70%" .center}

small network는 위의 그림에서 처럼 slinding-window에서 작동하여 저차원 feature을 매핑하기 때문에 *모든 공간적 위치에서 fully-connected layer가 공유된다.*

### 2-2) Anchor ###
&nbsp;&nbsp;sliding-window 위치마다 multiple region proposal을 계산하는 데 각 위치에서 최대 k개만큼 anchor를 사용하여 계산한다. 따라서 k에 따라 *reg* layer는 2k *cls* layer는 4k가 된다.

*cls* layer는 분류작업이므로 2k가 되고, *reg* layer는 anchor마다 4개의 좌표값이 있으므로 4k가 된다.

또한 anchor는 3개의 크기(512, 256, 192)와 3개의 비율(1:1, 1:2, 2:1)을 사용해 총 9개의 anchor를 논문에서 사용한다.

이 두 layer를 학습하여 물체의 정확한 bounding box(roi)를 추출하고 Roi pooling을 통해 각 region에 대한 feature map이 동일하게 고정된 크기로 생성된다.



## 3) Reference ##
[1] paper : https://arxiv.org/abs/1506.01497