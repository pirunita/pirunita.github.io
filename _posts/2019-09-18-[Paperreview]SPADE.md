---
layout: post
title:  "[Paper review] SPADE"
date:   2019-09-18 00:02:00 +0900
lang: ko
tags: DeepLearning Paper
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


## 3) Reference ##
[1] paper : 