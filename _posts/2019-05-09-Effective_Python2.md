---
layout: post
title:  "[Python] Effective Python(2) - List(1)"
date:   2019-05-09 09:00:00 +0900
lang: ko
tags: Python
---

## 시퀀스 슬라이싱 ##
&nbsp;&nbsp;파이썬은 슬라이싱으로 특정 subset에 접근할 수 있다. 가장 간단한 슬라이싱 대상은 list, str, bytes이다.
슬라이싱 문법은 somelist[start:end]이며, start 인덱스는 포함되고 end 인덱스는 제외된다.

~~~python
a = [1, 2, 3, 4, 5, 6, 7]
print(a[:4])
print(a[-4:])
print(a[3:-3])

>>>
1, 2, 3, 4
4, 5, 6, 7
4, 5
~~~

## 한 슬라이스에 start, end, stride를 함께 쓰지 않는다. ##

&nbsp;&nbsp; somelist[start:end:stride]처럼 슬라이스의 간격을 설정하는 특별한 문법도 있다. 이를 통해 시퀀스를 슬라이싱할 때 매 n번째 아이템을 가져올 수 있다.

~~~python
a = ['a', 'b', 'c', 'd', 'e', 'f']
odds = a[::2]
evens = a[1::2]
print(odds)
print(evens)

>>>
['a', 'c', 'e']
['b', 'd', 'f']
~~~
<br>

하지만 stride는 예상치 못한 동작을 하는데 파이썬에서 바이트 문자열을 역순으로 만드는 방법은 stride -1로 문자열을 슬라이싱 한다.
~~~python
x = b'mongoose'
y = x[::-1]
print(y)

>>>
b'esoognom'
~~~
위의 코드는 바이트 문자열이나 아스키 문자에는 잘 동작하지만 UTF-8 바이트 문자열로 인코드 된 유니코드 문자에서는 잘 동작하지 않는다.

stride의 다양한 문법을 보자.
~~~python
a = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h']
a[::2]      # ['a', 'c', 'e', 'g']
a[::-2]     # ['h', 'f', 'd', 'b'] 끝부터 시작해서 반대방향으로 두번 째 아이템 선택
a[2::2]     # ['c', 'e', 'g']
a[-2::-2]   # ['g', 'e', 'c', 'a']
a[-2:2:-2]  # ['g', 'e']
a[2:2:-2]   # []
~~~
슬라이싱에서 stride는 매우 혼란스러울 수 있다. 따라서 stride를 start, end 인덱스와 함께 사용하지 않는 것이 좋으며 만약 사용한다면 **양수 값을 사용하고 start와 end는 생략**한다. 만약 stride를 start, end와 같이 사용해야 한다면 stride 적용 결과를 변수에 할당하고 슬라이싱 결과를 다른 변수에 할당한다.

stride -> slicing

~~~python
b = a[::2]
c = b[1:-1]
~~~

만약 slicing -> stride를 하면 데이터의 얕은 복사본이 추가로 생긴다.

## map과 filter 대신 list comprehension을 사용 ##
&nbsp;&nbsp;파이썬에서는 한 리스트에서 다른 리스트를 만들 수 있는 list comprehension이 있다.
~~~python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
squares = [x**2 for x in a]
print(squares)

>>>
[1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
~~~
인수가 하나뿐인 함수를 적용하는 것이 아니면 간단한 연산은 list comprehension이 내장 함수 amp보다 명확하다. map은 계산에 필요한 lambda를 생성해야 한다.

~~~python
squares = map(lambda x: x**2, a)
~~~
<br>
또한 list comprehension을 사용하면 입력 리스트의 아이템을 잘라내서 그에 대응하는 출력을 삭제할 수 있다.

~~~python
even_squres = [x**2 for x in a if x % 2 == 0]

>>>
[4, 16, 36, 64, 100]
~~~
<br>
내장 함수 filter를 map과 같이 사용하면 읽기가 어려워진다.

~~~python
alt = map(lambda x: x**2, filter(lambda x: x % 2 == 0, a))
~~~
<br>

또한 dictionary, set에도 list comprehension에 해당하는 문법이 존재한다.
~~~python
color_channels = {'red': 1, 'green': 2, 'blue': 3}
rank_dict = {rank: color for color, rank in color_channels.items()}
color_len_set = {len(color) for color in color_channels.values()}
print(rank_dict)
print(color_len_set)

>>>
{1: 'red', 2: 'green', 3: 'blue'}
{3, 5, 4}
~~~

## list comprehension에서 표현식을 두 개 넘게 쓰지 않는다. ##
&nbsp;&nbsp;list comprehension에서는 다중 루프도 가능하다.
~~~python
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat = [x for row in matrix for x in row]
print(flat)

>>>
[1, 2, 3, 4, 5, 6, 7, 8, 9]
~~~
~~~python
squared = [[x**2 for x in row] for row in matrix]
print(squared)

>>>
[[1, 4, 9], [16, 25, 36], [49, 64, 81]]
~~~
하지만 여러 줄의 list comprehension은 그다지 짧지 않기 때문에 for문을 사용한다.

또한 list comprehension은 다중 if도 가능하다. 같은 for문에 여러 조건이 있으면 and표현식이 된다.
~~~python
a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
b = [x for x in a if x > 4 if x % 2 == 0]
c = [x for x in a if x > 4 and x % 2 == 0]
# b와 c는 동일
~~~

## list comprehension 시 제너레이터 표현식을 고려 ##
&nbsp;&nbsp;list comprehension은 각각의 값을 담은 새 리스트를 새로 생성하기 때문에 크면 메모리를 많이 소모한다. 파이썬은 이를 해결하기 위해 list comprehension과 제너레이터를 일반화한 generator expression을 제공한다. 제너레이터 표현식은 실행될 때 모두 메모리에로딩하지 않으며 표현식에서 하나에 한 아이템을 내주는 iterator로 평가된다.
~~~python
value = [len(x) for x in open('/tmp/my_file.txt')]
it = (len(x) for x in open('/tmp/my_file.txt'))

print(value)
print(next(it))
~~~
필요할 때 제너레이터 표현식에서 내장 함수 next로 반환받은 iterator를 한 번에 전진시키면 메모리 사용량을 걱정하지 않고 제너레이터 표현식을 사용하면 된다.

또한 제너레이터 표현식은 다른 제너레이터 표현식과 함께 사용할 수 있다.
~~~python
roots = ((x, x**0.5) for x in it)
print(next(roots))
~~~
위의 iterator를 전진시키면 내부의 iterator도 전진한다. 제너레이터를 연결하면 파이썬에서 매우 빨라진다.

