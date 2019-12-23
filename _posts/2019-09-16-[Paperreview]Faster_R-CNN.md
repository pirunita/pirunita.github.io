---
layout: post
title:  "[Paper review] Faster R-CNN"
date:   2019-09-16 00:02:00 +0900
lang: ko
tags: DeepLearning
---

## Faster R-CNN ##
<hr>

## 1) Introduction ##
&nbsp;&nbsp;이전의 object detection으로 제안된 **SPPnets**과 **Fast R-CNN**의 경우 region proposal을 만드는 데 걸리는 시간은 여전히 많이 걸린다.

region proposal을 생성하기 위해서 **selective search**를 사용하는데 이는 다음과 같은 문제를 갖는다.

1.  <u>2000개의 Region proposal을 생성</u>하며 bottleneck이 발생한다.

2. Region proposal을 만드는 데 <u>CPU를 사용한다.</u> 이는 sharing computation을 할 수 없음을 의미한다.

따라서 Faster R-CNN에서는 깊은 CNN을 이용하여 computing proposal 방법을 바꾼 것이 특징이라고 볼 수 있다.

Faster R-CNN에서는 기존의 Region proposal 대신 **Region Proposal Networks(RPNs)** 라는 네트워크를 도입하였다.

예상할 수 있는 것처럼 Region proposal을 생성하는 방법 자체를 CNN 내부의 네트워크 구조로 넣은 모델이며 Fast R-CNN, SPPNet에서 제안된 object detection networks의 convolutional networks와 정보를 공유할 수 있는 신경망이다.

RPNs의 특징은 다음과 같다.

1. data의 grid에서 각 위치마다 **region bounds**(bounding box라고 생각)와 **objectness scores**(roi pooling)를 동시에 구함으로써 같은 feature map을 공유하게 된다.

2. FCN와 비슷한 역할이 비슷하다.

3. object detection에 있어서 모두 neural network를 이용하기 때문에 end-to-end로 학습이 가능하다.

또한 RPNs는 보다 다양한 scale과 aspect ratio에서도 효율적으로 region proposals을 구하기 위해 **anchor boxes**라는 개념을 도입하였다. 이에 대한 동작은 Related work에서 설명한다.

기존의 Fast R-CNN의 object detection networks에서 RPNs를 합치기 위해 region proposal와 object detection 각각의 fine tuning을 바꾼 새로운 training scheme를 제안하였다.


## 2) Faster R-CNN

&nbsp;&nbsp;구조는 다음과 같다. 

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