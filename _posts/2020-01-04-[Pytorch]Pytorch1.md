---
layout: post
title:  "[PyTorch] Introducing the PyTorch Library"
date:   2020-01-04 12:00:00 +0900
lang: ko
tags: DeepLearning
---
<hr>

## PyTorch ##

### 1. What is PyTorch? ###
&nbsp;&nbsp;먼저 PyTorch는 Deep learning project를 빌드할 때 매우 유용한 python 기반의 library라고 할 수 있다. 접근성이 뛰어나고 다루기 쉬워 research community에선 일찍이 사용되어 왔으며 점점 넓은 분야로 활용되는 Deep Learning tool로 성장하고 있다.<br><br>
&nbsp;&nbsp;PyTorch도 Tensorflow처럼 다차원행렬인 **Tensor**를 핵심 자료구조로 사용하고 있다. PyTorch는 분산학습과 효율적인 data loading, 그리고 deep learning에 필요한 함수를 포함한 다양한 library를 포함한 패키지라고 볼 수 있다.

### 2. Why PyTorch? ###
&nbsp;&nbsp;Deep Learning은 다양하고 복잡한 task(machine translation, reinforcement learning, identifying object)를 수행할 수 있다. 이러한 task들을 수행하기 위해선 어떤 특정 문제에서도 효율적으로 작동하는 tool이 필요하며, 제 시간에 수많은 양을 학습시킬 수 있어야 한다. 또한 Input data의 불확실성을 정확하게 수행하는 훈련된 네트워크도 필요하다.<br><br>
&nbsp;&nbsp;PyTorch는 위에서 제시한 조건들을 충족시킬 수 있으며 간편하다. 학습, 사용, 확장, 디버깅도 간편하며 Pythonic하게 코드를 작성할 수 있다. 실제로 research에서 development, production까지 광범위하게 사용될 수 있는 데, 특히 model을 deploy할 때 python runtime의 오버헤드 없이 PyTorch만의 유연성을 유지하며 Python에 의존하지 않고도 high-performance C++ runtime를 갖추고 있다.(*실제로도 꽤 빠른 편이라고 들었는 데 어떻게 병렬처리 하는 지는??*)<br><br>
&nbsp;&nbsp;Deep Learning framework 중 유명한 것은 **Tensorflow**가 있는 데 <u>지연 수행(deferred execution)</u>과 유사한 **graph mode**를 사용하여 graph를 먼저 다 만들어 놓고 연산을 수행한다. 하지만 PyTorch는 코드가 평가될 때 노드 별로 컴퓨팅 연산 그래프가 작성되는 **define-by-run dynamic graph engine**을 사용하여 필요에 따라 node를 바꿔 연산을 쉽고 직관적으로 변경할 수 있으며 즉시 수행(immediate execution, **eager mode**)<br><br>
&nbsp;&nbsp;Tensorflow는 production에 robust한 pipeline을 가져 광범위한 산업분야에 사용된다. PyTorch는 사용하기 쉬워 research나 교육에 많이 사용한다.<br><br>

### 3. PyTorch has the batteries included ###
&nbsp;&nbsp;PyTorch를 구성하는 주 요소를 알아 본다.
1. PyTorch의 어원이 Python에서 나온 것처럼 보이지만 대부분 C++과 CUDA로 작성되어 있으며 NVIDA GPUs에서 병렬 처리를 수행할 때는 C로 컴파일된다. 이는 model deploy가 쉽기 때문이다. 하지만 PyTorch는 Python과 상호작용하여 모델을 짜고 학습시키고 문제를 해결하기 때문에 Python으로 충분히 할 수 있다.
2. PyTorch는 tensors라 불리는 다차원 행렬을 제공하는 library이며, **torch** module이 제공하는 다양한 연산 라이브러리가 존재한다. 이 때 tensor와 연산들은 CPU나 GPU에서 실행할 수 있다. GPU로 연산을 수행할 시 속도가 굉장히 빠르고 이를 위해 PyTorch에서는 별도로 추가적인 함수를 수행하지 않는다.

&nbsp;&nbsp;PyTorch는 Neural networks를 학습하고 빌드하는 데 필요한 building blocks를 제공한다. Neural networks를 빌드하기 위한 PyTorch modules는 **torch.nn**을 사용하며 공통적으로 사용하는 neural network layer와 여러 구조적인 component를 제공한다.(Fully connected layers, Convolutional layers, Activation functions, loss functions ..)<br><br>

&nbsp;&nbsp;Data loading과 hadnling을 위한 유틸리티는 **torch.util.data.** 에서 찾을 수 있다. 사용할 수 있는 두 개의 main class가 있다.
* Dataset: custom data와 Tensor 사이의 bridge 역할
* DataLoader: Dataset으로부터 data를 load하는 child process를 만들어 training loop를 바로 수행할 수 있도록 준비해 두는 역할

&nbsp;&nbsp;model을 학습시키기 위해 필요한 것은 다음과 같다.
1. a source of training data
2. an optimizer to adapt the model to the training data
3. hardware

&nbsp;&nbsp;간단한 연산이 요구될 때 model은 local CPU나 single GPU에서 수행되어 training loop가 data를 얻을 때 연산이 즉시 시작할 수 있다. 하지만 multi GPU로 학습하고자 할 때 모델을 학습하기 위한 리소스들을 분산해야 한다. 이럴 때는 **torch.nn.DataParallel, torch.distributed**를 수행하여 추가적인 hardware를 가용할 수 있다.<br><br>

&nbsp;&nbsp;Training data로부터 model을 학습하고 나온 결과를 얻을 때, **torch.optim**는 modle update의 standard way를 제공하여 output이 training data가 요구한 답과 비슷하게 만든다.

<hr>
Eli Stevens, Luca Antiga. Deep Learning with PyTorch

