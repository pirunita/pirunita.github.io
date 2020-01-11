---
layout: post
title:  "[PyTorch] 4. It starts with a tensor(3)"
date:   2020-01-11 12:00:00 +0900
lang: ko
tags: DeepLearning
---
<hr>

## PyTorch ##

### 4. Numeric types ###
&nbsp;&nbsp;Tensor도 numpy array의 dtype argument처럼 numerical data type을 가지게 된다.

| torch dtype                  | numerical data type                     |
|------------------------------|-----------------------------------------|
| torch.float32 or torch.float | 32-bit, floating-point                  |
| torch.float64 or torch.double| 64-bit, double_precision floating-point |
| torch.float16 or torch.half  | 16-bit, half-preicision floating-point  |
| torch.int8                   | Signed 8-bit integers                   | 
| torch.int16 or torch.short   | Signed 16-bit integers                  | 
| torch.int32 or torch.int     | Signed 32-bit integers                  |
| torch.int64 or torch.long    | Signed 64-bit integers                  |

tensor에 적절한 numeric type을 부여하기 위해 argument로 **dtype**을 넣어 정한다. 또한 dtype에 대해 찾으려면 예시는 다음과 같다.

~~~ python
double_points = torch.ones(10, 2, type=torch.double)
short_points = torch.tensor([[1, 2], [3, 4]], dtype=torch.short)
short_points.dtype
~~~
~~~
>>>
torch.int16
~~~

또한 **casting method**를 이용하여 적절한 type으로 casting하는 다양한 **tensor-creation function**을 쓸 수 있다. method로는 **to**, **type**을 사용할 수 있다.
~~~ python
double_points = torch.zeros(10, 2).double()
short_points = torch.ones(10, 2).short()

double_points = torch.zeros(10, 2).to(torch.double)
short_points = torch.ones(10, 2).to(dtype=torch.short)

points = torch.randn(10, 2)
short_points = points.type(torch.short)
~~~

### 5. Indexing tensors ###
&nbsp;&nbsp;Tensor는 python의 기본 list처럼 index들을 **slicing** 할 수 있다. range를 사용함으로써 PyTorch도 강력한 indexing을 가지며 이를 **advanced indexing**이라고 한다.
~~~ python
# list
some_list = list(range(6))
some_list[:]            # All elements in the list
some_list[1:4]          # From element 1 inclusive to element 4 exclusive
some_list[1:]           # From element 1 inclusive to the end of the list
some_list[:4]           # From the start of the list to element 4 exclusive
some_list[:-1]          # From the start of the list to one before the last element
some_list[1:4:2]        # From element 1 inclusive to element 4 exclusive in steps of 2

# Tensor
points[1:]              # All rows after first, implicitly all columns
points[1:, :]           # All rows after first, all columns
points[1:, 0]           # All rows after first, first column
~~~

### 6. Numpy interoperability ###
&nbsp;&nbsp;PyTorch의 tensor는 Numpy array로 변환될 수 있어서 Numpy 배열을 중심으로 구축된 더 넓은 Python ecosystem의 방대한 기능들을 효율적으로 사용할 수 있다. 그리고 Numpy array는 Python buffer protocol이 작동하는 storage system덕분에 **zero-copy interoperability**가 가능하다.

~~~ python
points = torch.ones(3, 4)
points_np = points.numpy()
points_np
~~~
~~~
>>>
array([[1., 1., 1., 1.],
       [1., 1., 1., 1.],
       [1., 1., 1., 1.]], dtype=float32)
~~~

다음과 같이 같은 size, shape, numerical type의 Numpy 다차원 배열을 반환하며, *이 때 반환되는 array는 tensor storage와 기본 buffer를 공유한다.* 즉, **numpy method**는 CPU RAM에서 cost가 꼭 소모되지 않고도 Numpy array에서 originating tensor로 바꿀 수 있다. 또한 GPU에 할당된 tensor가 있다면, PyTorch는 CPU에 할당되는 Numpy array로 복사하게 된다. 

PyTorch tensor에서 Numpy array로 바꾸는 것은 다음과 같다.

~~~ python
points = torch.from_numpy(points_np)
~~~

### 7. Seriallizing tensors ###
&nbsp;&nbsp;PyTorch는 tensor object를 직렬화 하기 위해 **pickle**을 사용하고, storage에 serialization 코드를 사용한다. tensor를 저장 해 보자!

~~~ python
""" Saving points """
torch.save(points, '../data/ourpoints.t')

# or

with open('./data/ourpoints.t', 'wb') as f:
    torch.save(points. f)

""" Loading points back """
points = torch.load('./data/ourpoints.t')

# or

with open('./data/ourpoitns.t', 'rb') as f:
    points = torch.load(f)
~~~

이러한 테크닉은 tensor를 저장하고 원할 때 PyTorch에서 불러올 수 있게 되지만 프로젝트에선 자주 사용되지는 않는다. 하지만 PyTorch를 이미 다른 library에 의존하는 시스템에 도입하려 할 때 사용될 수 있다. 직렬화된 다차원 행렬을 지원하는 HDF5 format과 library를 사용하는 예를 들어보자.

~~~ python
import h5py

f = h5py.File('./data/ourpoints.hdf5', 'w')
dataset = f.create_dataset('coords', data=points.numpy())
f.close()
~~~

여기서 **'coords'**는 HDF5 file의 **key** 이다. 여기서 우리는 dataset을 index할 수 있으며 우리가 원하는 element에 **only-access**가 가능하다.

~~~ python
f = h5py.File('./data/ourpoints.hdf5', 'r')
dataset = f['coords']
last_points = dataset[1:]
~~~

여기서는 파일을 열거나 Dataset이 필요할 때 Data가 로드되지 않고, dataset[1:]을 요청할 때까지 데이터는 디스크에 남아 있었다. 이 때 h5py는 이 두 column에 access해서 Numpy array처럼 동작하고 동일한 API를 갖는 해당 dataset의 영역을 encapsulation하는 **Numpy 배열의 유사 객체를 반환**한다.<br>
반환된 객체는 **torch.from_numpy** 함수에 전달하여 tensor를 얻을 수 있고 Data는 tensor storage로 복사된다.

~~~ python
last_points = torch.from_numpy(dataset[1:])
f.close()
~~~

<hr>
Eli Stevens, Luca Antiga. Deep Learning with PyTorch



