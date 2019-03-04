---
layout: post
title:  "M2E-Try On Net"
date:   2018-12-14 21:47:00 +0900
lang: ko
tags: DeepLearning GAN Virtual-Try-On
---
# M2E-Try On Net: Fashion from Model to Everyone #

##### Zhonghua Wu, et al. "M2E-Try On Net: Fashion from Model to Everyone" arXiv: 1811.08599 (2018). #####
<hr>

## 1. Introduction ##
&nbsp;&nbsp;VITON처럼 3D 정보 없이 virtual-try-on system을 만들고 싶다. M2E에서 하고자 하는 것은 요약하면 다음 3가지와 같다.
<br>
![]({{ site.url }}/assets/img/M2E-TON/01.png){: width="70%" height="70%" .center}
* M2E-TON이라 불리는 virtual Try-On Network는 원하는 모델의 옷을 임의의 사람 사진에 자동으로 바꿔준다. 이는 M2E-TON가 디자인 한 sub-network들을 통해 texture을 보존하면서 사람의 pose에 따라 잘 정립된 real한 이미지를 만들어 줄 것이다. 특히 M2E-TON은 <u>product image가 필요 없다는 것</u>이 큰 특징이다.

* M2E-TON에서는 3가지 sub-network를 제안한다.
    + model image에서 target person pose로 정렬되는 **pose alignment network(PAN)** 도입
    + 옷의 특징을 더욱 끌어내기 위해 pose가 정렬 된 이미지에 변형된 clothes textures를 추가하는 **Texture Refinement Network(TRN)**을 도입
    + 원하는 옷을 target person image에 맞추기 위한 **Fitting Network(FTN)**을 도입

* target person P와 원하는 model clothes인 P'는 unpair한 학습 데이터이기 때문에 **unsupervised learning & self-supervised learning**의 <u>hybrid learning framework</u>을 사용한다.

<br>

## 2. Related Works ##
&nbsp;&nbsp;M2E-TON은 human parsing & analysis, person image generation, virtul try-on, fashion dataset과 관련이 있다.

### 2.1. Human Parsing and Human Pose Estimation ###
&nbsp;&nbsp;더 정확한 pose estimation을 위해 **DensePose**를 이용한다. DensePose는 각 픽셀마다 dense pose point를 mapping시켜 human pose estimation을 얻는 방법이다. M2E-TON에서는 clothes region warping과 pose alignment을 위해 **estimated dense poses**를 활용한다.
<br>

### 2.2. Person Image Generation and Virtual Try-On ###
&nbsp;&nbsp;이전에 시도되었던 virtual try on은 product image를 input으로 넣는 방식이었다.<br>
하지만 M2E-TON은 identity와 pose를 유지하면서 model pearson에서 target person으로 자연스럽게 변할 수 있다.

### 2.3. Fashion Datasets ###
* Deep Fashion dataset : clothes attribute prediction과 landmark detection을 위한 fashion dataset
* MVC dataset : invariant clothing retrieval and attribute prediction

<br>

## 3. Approach ##
![]({{ site.url }}/assets/img/M2E-TON/02.png){: width="70%" height="70%" .center}
### 3.1. Pose Alignment Network (PAN) ###

### 3.2. Texture Refinement Network (TRN) ###

### 3.3. Fitting Network (FTN) ###

### 3.4. Unpair-Pair Joint Training ###

### 3.5. Loss Functions ###