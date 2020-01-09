---
layout: post
title:  "[PyTorch] 3. It starts with a tensor(2)"
date:   2020-01-10 12:00:00 +0900
lang: ko
tags: DeepLearning
---
<hr>

## PyTorch ##

### 3. Size, storage offset, strides ###
&nbsp;&nbsp;Storage에 index를 붙여주기 위해서 tensor는 storage에서 몇 가지 정보를 포함하고 결정한다. 이는 size, storage offset, stride이며 다음과 같이 도식화할 수 있다.
![]({{ site.url }}/assets/img/pytorch/3_1.png){: width="85%" height="85%" .center}

* Size: Tensor가 나타내는 각 차원마다 얼만큼 요소가 있는 지 나타내는 tuple이다.
* Storage offset: tensor에서 첫 번째 element와 일치하는 storage에 저장된 index이다.
* Stride: 각 차원에 따라 다음 element를 얻을 때 skip하기 위해 필요한 storage에서의 element 수 이다.

~~~ python
points = torch.tensor([1.0, 4.0], [2.0, 1.0], [3.0, 5.0])
second_point = points[1]
second_point.storage_offset()
second_point.size()
second_point.shape
~~~
~~~
>>>
2
torch.Size([2])
torch.Size([2])
~~~

여기서 size()나 shape의 결과는 storage에서 offset 2를 갖는다. 이는 second_point에 points[1] 즉 두 번째 인덱스를 가리키기 때문이다.

~~~ python
points.stride()
second_point.stride()
~~~
~~~
(2, 1)
(1,)
~~~

Tensor를 전치(transpose)시키거나 subtensor를 추출할 때 메모리를 재할당하는 것은 그 연산비용이 많이 들기 때문에 **size, storage offset, stride**를 이용하여 새로운 tensor object를 구성하게 된다. 따라서 subtensor를 바꿀경우 원래의 tensor도 바뀌는 **side effect**를 갖는다.

~~~ python
points = torch.tensor([1.0, 4.0], [2.0, 1.0], [3.0, 5.0])
second_point = points[1]
second_point[0] = 10.0
points
~~~
~~~
>>>
tensor([[1., 4.],
        [10., 1.],
        [3., 5.]])
~~~

하지만 이는 원래의 tensor 값을 보장하지 못하기 때문에 subtensor는 **clone**을 통해 무결성을 보장한다. 

~~~ python
points = torch.tensor([1.0, 4.0], [2.0, 1.0], [3.0, 5.0])
second_point = points[1].clone # clone 추가
second_points[0] = 10.0
points
~~~
~~~
>>>
tensor([[1., 4.],
        [2., 1.],
        [3., 5.]])
~~~

이번에는 tensor를 transpose 시키고 원래의 tensor와 storage를 공유하는 지 비교 해 본다.

~~~ python
points = torch.tensor([1.0, 4.0], [2.0, 1.0], [3.0, 5.0])
points_t = points.t()
points
points_t
id(points.storage()) == id(points_t.storage())
~~~
~~~
>>>
tensor([[1., 4.],
        [2., 1.],
        [3., 5.]])
tensor([[1., 2., 3.],
        [4., 1., 5.]])
True
~~~

True가 나오는 것으로 보아 두 tensor는 storage를 공유한다고 볼 수 있다. 이번에는 shape와 stride를 비교 해 본다.

~~~ python
points.stride()
points_t.stride()
~~~
~~~
(2, 1)
(1, 2)
~~~

이 때 transpose 연산은 다음과 같이 이루어지며 새로운 메모리는 할당되지 않고 오직 stride를 바꿔 새로운 tensor를 만드는 것이다.

![]({{ site.url }}/assets/img/pytorch/3_2.png){: width="85%" height="85%" .center}

또한 PyTorch에서 transpose는 matrix뿐만 아니라 multidimensional aray에서도 적용된다.

~~~ python
some_tensor = torch.ones(3, 4, 5)
some_tensor_t = some_tensor.transpose(0, 2)
some_tensor.shape
some_tensor_t.shape
some_tensor.stride()
some_tensor_t.stride()
~~~
~~~
>>>
torch.Size([3, 4, 5])
torch.Size([5, 4, 3])
~~~
(2, 1)
(1, 2)
(20, 5, 1)
(1, 5, 20)
~~~

또한 Contiguous tensor는 storage에서 건너 뛰지않고 원래의 tensor에서 순차적으로 방문할 수 있다. 이 때 tensor가 contiguous한 지 아닌 지 method를 사용해서 알 수 있다.

~~~ python
points.is_contiguous()
points_t.is_contiguous()
~~~
~~~
True
False
~~~

또한 noncontiguous tensor로부터 새로운 **contiguous tensor**를 얻기 위해 **contiguous method**를 사용한다.

~~~ python
points = torch.tensor([1.0, 4.0], [2.0, 1.0], [3.0, 5.0])
points_t = points.t()
points_t
~~~
~~~
>>>
tensor([[1., 2., 3.], 
        [4., 1., 5.]])
~~~
~~~ python
points_t.storage()
points_t.stride()
~~~
~~~
>>>
1.0
4.0
2.0
1.0
3.0
5.0
[torch.FloatStorage of size 6]
(1, 2)
~~~

transpose한 tensor에서 contiguous 결과는 다음과 같다.

~~~ python
points_t_cont = points_t.contiguous()
points_t_cont
points_t_stride()
points_t_cont.storage()
~~~
~~~
>>>
tensor([[1., 2., 3.],
        [4., 1., 5.]])
(3, 1)
1.0
2.0
3.0
4.0
1.0
5.0
[torch.FloatStorage of size 6]
~~~

여기서 중요한 점은 새로운 storage에서 row by row로 놓여진 기존의 storage의 element가 reshuffle 된다는 것을 기억하자

<hr>
Eli Stevens, Luca Antiga. Deep Learning with PyTorch



