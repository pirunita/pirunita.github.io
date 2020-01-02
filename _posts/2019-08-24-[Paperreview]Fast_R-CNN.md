---
layout: post
title:  "[Paper review] Fast R-CNN"
date:   2019-08-24 00:02:00 +0900
lang: ko
tags: DeepLearning Paper
---

## Fast R-CNN ##
<hr>

## 1) Introduction ##
&nbsp;&nbsp;전에 리뷰한 R-CNN에는 3가지에 문제가 있다.
1. 3단계 모델을 거친 pipeline 구조<br>
Region-proposal을 추출하기 위해 Selective search 알고리즘, feature vector을 얻기 위한 CNN, 그리고 feature vector로 분류하는 SVM classifier, 더 정확한 bounding-box를 얻기위한 regressor
2. Training에 자원과 시간이 많이 소모<br>
각각 이미지에서 얻은 2000개의 region-proposal로부터 CNN을 통해 얻은 feature vector로 SVM과 bounding-box regressor를 훈련할 때 디스크에 이 data들을 모두 쓰면서 deep network를 training한다.
3. 느린 수행 속도<br>
역시나 각각의 test image에서 region proposal, feature vector을 얻는 데 시간이 걸린다.
4. 원치않는 이미지의 왜곡과 손실
CNN의 input을 맞추기 위해 warping 및 cropping이 일어나 정보의 손실 발생

앞서 b-box 사이에 겹치는 영역을 따로따로 CNN에 통과시키는 것은 비용적으로 낭비이다. 그래서 제안된 것이 **SPPnets**이었다. 

### Spatial Pyramid Pooling networks(SPPnets) ###
&nbsp;&nbsp;앞서 R-CNN이 비용 낭비가 심하므로 R-CNN의 속도를 개선하기 위해 computation을 공유하도록 했다. 즉 SPPnets는 전체 *이미지에서 convolutional feature map을 먼저 계산하고 feature map으로부터 pooling으로 얻은 feature vector를 분류하는 방법*이다. 이러면 feature map이라는 계산을 공유하게 된다.

![]({{ site.url }}/assets/img/rcnn/fastrcnn1.png){: width="70%" height="70%" .center}

이 때 사용된 **Spatial pyramid pooling**은 feature map으로부터 다양한 사이즈로 영역을 분할하고 max-pooling을 통해 feature를 얻고 이들을 concatenation 시켜 fixed-length representation을 생성한다. 이를 FC로 넣어 training과 inference 속도를 개선하였다. training은 3배정도 빨라지고 수행 속도도 10배에서 100배까지 빨라졌다.

R-CNN은 CNN이 도는 횟수가 많았으나 SPPnet은 CNN이 한 번만 수행되어 빨라질 수 있었으며 feature map에서 정보를 추출한다는 특징이다.

또한 중요한 건 pyramid layer로 부터 얻는 representation(feature vector)이므로 이것만 미리 정하면 FC의 input으로 넣을 수 있으며 최초의 input 크기에 제한을 받지 않게 된다. 기존의 AlexNet과 같은 신경망의 입력 size가 고정되는 것은 FC이기 때문이다.

#### Drawback ####
하지만 SPPnets 역시 multi-stage pipeline 구조라는 한계를 지니며(feature 추출, network fine-tuning, SVM 학습, fitting b-box regressor) feature들을 모두 disk에 쓰여져야 한다.

그리고 R-CNN과 다르게 spatial pyramid pooling을 예측하는 convolutional layer는 update를 할 수 없다. **(Q1)**

그렇다면 이제 문제 해결 목표를 다시 정의 해 보자.

1. R-CNN, SPPnet보다 정확한 object-detection
2. multi-task loss를 이용한 single-stage training
3. 학습 시 모든 network layer update가 가능해야 함.
4. feature를 disk에 쓰는 과정이 없어야 함.

## 2) Fast R-CNN architecture and training
&nbsp;&nbsp;구조는 다음과 같다. 

![]({{ site.url }}/assets/img/rcnn/fastrcnn2.png){: width="70%" height="70%" .center}

1. ConvNet을 거쳐 max pooling layer을 통해 conv feature map을 얻는다. 이 때 이미지와 region-proposal을 한 번에 넣는다. regional-proposal을 얻기 위해 selectvie search를 사용한다.
2. feature map으로 부터 fixed-length feature vector를 얻기 위해 각 object proposal의 **RoI(Region of Interest) pooling layer**을 거친다.
3. 얻은 feature vector에 각각 FC와 연결된 Softmax classifier & b-box regressor 두 개의 브런치 레이어를 먹인다. 
    * Softmax classifier: object class와 배경 추정
    * b-box regressor: object class 좌표를 추정

R-CNN은 이미지를 warping했다면 Fast R-CNN은 *feature map*을 고정되게 warping해야 한다. 이를 위해 **RoI pooling layer**를 사용한다.

### 2-1) The RoI pooling layer ###
![]({{ site.url }}/assets/img/rcnn/fastrcnn3.png){: width="70%" height="70%" .center}


&nbsp;&nbsp;RoI layer는 위에서 설명한 SPPnets의 가장 단순한 케이스이다. SPPnet이 여러 size의 bin으로 영역을 나눴다면 RoI layer는 하나로 정해진 bin의 갯수로 RoI에 대해 한 번만 pooling을 진행한다. 이를 통해 fixed-length feature vector를 얻는다.

### 2-2) Fine-tuning for detection ###
![]({{ site.url }}/assets/img/rcnn/fastrcnn4.png){: width="70%" height="70%" .center}


&nbsp;&nbsp;Fast R-CNN의 Network의 weight들을 갱신하기 위해 back-propagation을 한다. 그림과 같이 모든 layer에 대해 학습이 가능한 1 stage pipeline으로 구성되어 error-backProp이 가능해 정확도 개선이 이루어졌다.

### 2-3) Region-wise sampling ###
&nbsp;&nbsp;Training time을 줄이기 위해 **region-wise sampling** 방법을 사용하였다. R-CNN의 경우 모든 region-proposal을 동일한 크기(224 × 224)로 바꾸지만 Fast R-CNN은 원본 그대로 사용하여 연산량이 증가하는 최악의 경우가 생긴다. 이를 피하기 위해 **hierarchical sampling** 방식을 사용한다.

기존의 R-CNN, SPPnet에서는 한 mini-batch 당 128개의 data에서 RoI를 무작위로 선택하였다. Fast R-CNN에서는 2개의 input으로부터 128개의 RoI(하나 당 64개)를 정함으로써 이미지의 RoI는 연산과 메모리를 공유하게 된다.**(Q2)**




## 3) Reference ##
[1] paper : https://arxiv.org/abs/1504.08083

[2] https://tensorflow.blog/2017/06/05/from-r-cnn-to-mask-r-cnn/

[3] https://nittaku.tistory.com/273

[4] https://m.blog.naver.com/PostView.nhn?blogId=laonple&logNo=220776743537&proxyReferer=https%3A%2F%2Fwww.google.com%2F