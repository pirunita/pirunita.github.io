---
layout: post
title:  "[PyTorch] 5. It starts with a tensor(4) - GPU execution & API"
date:   2020-01-12 12:00:00 +0900
lang: ko
tags: DeepLearning
---
<hr>

## PyTorch ##

### 8. Moving tensors to the GPU ###
&nbsp;&nbsp;모든 Torch tensor는 GPU에서 빠르고 대규모 병렬 연산을 수행할 수 있다. Tensor에서 수행되는 모든 작업은 PyTorch와 함께 제공되는 GPU 관련 루틴에 의해 수행된다. 
Dtype 외에도 PyTorch 텐서에는 컴퓨터에 Tensor가 배치되는 device 개념이 있는데, 생성자에 해당하는 인수를 지정하여 GPU에서 tensor를 만드는 방법은 다음과 같다.

~~~ python
points_gpu = torch.tensor([[1.0, 4.0], [2.0, 1.0], [3.0, 4.0]], device='cuda')
~~~

혹은 CPU에서 생성된 tensor를 **to** method를 사용함으로써 GPU에 tensor를 복사할 수 있다.

~~~ python
points_gpu = points.to(device='cuda')
~~~

위 코드는 같은 numerical data를 갖는 새로운 tensor를 반환하지만 GPU의 RAM에 저장된다. 이제 Data가 GPU에 지역적으로 저장되었으면, *tensor에서 수학적 연산을 빠르게 수행* 할 수 있다. 이 때 CPU든 GPU든 같은 API를 쉽게 사용할 수 있으며 여러 GPU를 사용할 경우 GPU를 지정하여 tensor를 할당할 수 있다.

~~~ python
points = 2 * points                             # Multiplication performed on the CPU
points_gpu = 2 * points.to(device='cuda:0')     # Multiplication performed on the GPU
~~~

이 때 주의할 점은 연산이 수행될 때 **points_gpu tensor**는 다시 CPU로 되돌아갈 수 없다. 따라서 한 번 GPU에 복사된 tensor는 계속해서 GPU에서 수행되게 된다. 이는 다음과 같은 수행을 따른다.

1. **points tensor**가 GPU에 복사된다.
2. 새로운 tensor가 GPU에 할당되고 곱셈 연산의 결과를 저장한다.
3. GPU tensor의 결과값이 반환된다.

따라서 다시 CPU로 tensor를 되돌아가게 하려면 **to method**에서 **cpu** argument를 사용한다.

~~~ python
points_gpu = points_gpu + 4                 # 계속해서 GPU에서 연산 수행

points_cpu = points_gpu.to(device='cpu')
points_gpu = points.cuda()                  # Default는 GPU:0
points_gpu = points.cuda(0)
points_cpu = points_gpu.cpu()
~~~
 

### 9. The tensor API ###

online documentation: <a src='http://pytorch.org/docs'>http://pytorch.org/docs</a>

&nbsp;&nbsp;대부분의 tensor 작업은 **torch** module에서 사용 가능하며 tensor object의 method이다. 예를 들어, 이전에 접했던 transpose 기능은 torch module에서 사용 가능하다.

~~~ python
# 1.
a = torch.ones(3, 2)
a_t = torch.transpose(a, 0, 1)

# 2.
a = torch.ones(3, 2)
a_t = a.transpose(0, 1)
~~~

위의 두 방법에는 차이가 없으며 상호적으로 이용할 수 있다. 하지만 몇 가지 method는 ***tensor object**의 method로만 존재한다. zero_ 처럼 name에서 따라오는 underscore로 인식할 수 있다. 이는 새로운 출력 tensor를 작성하는 대신 input을 수정하여 method가 제자리에서 작동함을 나타낸다.

여기서 zero_ method는 입력의 모든 요소를 0으로 만들며 밑줄이 없는 method는 **source tensor**를 변경하지 않고 새로운 tensor를 반환한다.

~~~ python
a = torch.ones(3, 2)
a.zero_()
a
~~~
~~~
>>>
tensor([[0., 0.],
        [0., 0.], 
        [0., 0.]])
~~~

또한 online documentation에서는 tensor operation을 그룹으로 나눈다.

* Creation ops
    - Tensor를 형성하는 연산
    - **ones, from_numpy**
* Indexing, slicing, joining, mutating ops
    - shape, stride, tensor 내용을 바꾸는 연산
    - **transpose**
* Math ops
    - 연산을 통해 tensor의 내용을 조작하는 연산
    - Pointwise ops
        - 함수를 각 요소에 독립적으로 적용하여 새로운 tensor를 얻는 함수
        - **abs, cos**
    - Reduction ops
        - 반복해서 집계 값을 계산하는 함수
        - **mean, std, norm**
    - Comparison ops
        - tensor에 대해 numerical predicate를 계산하는 함수
        - **equal, max**
    - Spectral ops
        - Frequency 영억에서의 변환 및 작동을 위한 함수
        - **stft, hamming_window**
    - Other ops
        - Vector로 수행하는 특별한 함수
        - **vector: cross**
        - **matrix: trace**
    - BLAS & LAPACK ops
        - scalar, vector-vector, matgrix-vector, matrix-matrix 연산에 의한 BLAS(Basic Linear Algebra Subprograms) specifaction 함수
* Random sampling ops
    - 확률분포로부터 무작위로 value를 생성하는 함수
    - **randn, normal**
* Parallelism ops
    - 병렬 CPU 연산을 수행할 때 threads의 수를 조정할 수 있는 함수
    - **set_num_threads**
 
<hr>
Eli Stevens, Luca Antiga. Deep Learning with PyTorch



