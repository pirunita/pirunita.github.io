---
layout: post
title:  "[PyTorch] It starts with a tensor"
date:   2020-01-07 00:00:00 +0900
lang: ko
tags: DeepLearning
---
<hr>

## PyTorch ##

&nbsp;&nbsp;Network는 **floating-point numbers**를 이용하여 information을 다루기 때문에 network가 이해할 수 있는 적절한 표현으로 data를 encoding하고 다시 우리가 이해할 수 있는 표현으로 decoding하여 output을 내야 한다.<br>

&nbsp;&nbsp;Data의 어떤 form에서 다른 form으로 transformation은 Deep neural network에 의해 학습된다. 이는 연속된 intermediate representation이 진행되는 단계 사이에 data가 부분적으로 transform된다고 볼 수 있다. 예를 들어 image recognition에서 early representation으로는 edge detection, texture(fur)이 될 수 있고, Deeper representation에서는 더욱 복잡한 구조(눈, 코, 입)를 찾아낼 수 있다.

<hr>
Eli Stevens, Luca Antiga. Deep Learning with PyTorch

