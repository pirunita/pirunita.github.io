---
layout: post
title:  "[PyTorch] 2. It starts with a tensor"
date:   2020-01-05 12:00:00 +0900
lang: ko
tags: DeepLearning
---
<hr>

## PyTorch ##

&nbsp;&nbsp;Network는 **floating-point numbers**를 이용하여 information을 다루기 때문에 network가 이해할 수 있는 적절한 표현으로 data를 encoding하고 다시 우리가 이해할 수 있는 표현으로 decoding하여 output을 내야 한다.<br>

&nbsp;&nbsp;Data의 어떤 form에서 다른 form으로 변환하는 것은 Deep neural network에 의해 학습된다. 이는 연속된 intermediate representation이 진행되는 단계 사이에 data가 부분적으로 transform된다고 볼 수 있다. <br>예를 들어 image recognition에서 early representation으로는 edge detection, texture(fur)이 될 수 있고, Deeper representation에서는 더욱 복잡한 구조(눈, 코, 입)를 찾아낼 수 있다.<br>
앞선 예시처럼 일반적으로 이러한 intermediate representations은 <u>Input을 특징화하고</u>, <u>Data에서 구조를 파악</u>하며 <u>Neural network의 output을 묘사</u>하게 된다.
![]({{ site.url }}/assets/img/pytorch/2_1.png){: width="85%" height="85%" .center}
<br><br>
&nbsp;&nbsp;Intermediate representation을 위해 Data를 적절하게 floating-point input으로 바꿀 필요가 있으며 PyTorch에서는 기본 자료구조로 **tensor**를 이용하게 된다. 물론 tensor는 NumPy, SciPy, Scikit-learn, Pandas와 같은 라이브러리에서도 사용된다.


### 1. Tensor fundamentals ###
tensor는 torch의 function을 이용하여 생성할 수도 있다. 또한 임의로 텐서를 바꿀 수도 있다. 하지만 딥러닝에선 잘 안쓰인다.
~~~ python 
import torch
a = torch.ones(3)
a
a[1]

float(a[1])
a[2] = 2.0
a
~~~
~~~
>>>
tensor([1., 1., 1.])
tensor(1.)
1.0
tensor([1., 1., 2.])
~~~

2차원 tensor도 만들어 보자
~~~ python
points = torch.tensor([1.0, 4.0], [2.0, 1.0], [3.0, 5.0])
points
points.shape
~~~
~~~
>>> 
tensor([1., 4.], [2., 1.], [3., 5.])
torch.Size([3, 2])
~~~

size를 이용하여 **zeros**나 **ones**로 tensor를 만들 수 있고, **FloatTensor**로 만들 수도 있다.
~~~ python
points = torch.zeros(3, 2)
points
points = torch.FloatTensor([1.0, 4.0], [2.0, 1.0], [3.0, 5.0])
points
points[0, 1]
points[0]
~~~
~~~
tensor([0., 0.],
       [0., 0.],
       [0., 0.])

tensor([1., 4.],
       [2., 1.],
       [3., 5.])

tensor(4.)
tensor([1., 4.])
~~~

위에서 output으로 나온 결과는 **다른 tensor**이지만 새롭게 메모리를 할당받은 것은 아니다. 만약 tensor가 엄청나게 커지면 process가 부족해 지기 때문이다. 다만 같은 data 내에서 다른 **view**를 갖게 되는 것이다.

### 2. Tensors and storages ###
&nbsp;&nbsp;Tensor 내에서 value들은 contiguous 메모리에 할당 되고 이는 **torch.Storage** 인스턴스에 의해 관리된다.

<hr>
Eli Stevens, Luca Antiga. Deep Learning with PyTorch



