---
layout: post
title:  "STL Container"
date:   2019-04-16 12:00:00 +0900
lang: ko
tags: ProblemSolving
---

*이번부터는 cpp로 PS를 하기 위해 필요한 STL 컨테이너(docker의 컨테이너 아님)의 내부 자료구조와 동작을 간략하게 보고 PS에 필요한 것만 공부한다.*
<br>
<br>


&nbsp;&nbsp;C++ 표준 라이브러리는 특별한 컨테이너들(**stack**, **queue**, **priority queue** 등..)을 제공하는 데, 이들에는 공통적인 특징과 동작들이 있다.
* 컨테이너의 공통적인 특징
1. 모든 컨테이너는 "값 의미론"을 제공하기 때문에, 컨테이너에 값이 삽입될 경우 내부적으로 복사본을 생성한다. 따라서 모든 STL 컨테이너의 요소들은 복사되어질 수 있어야 한다.
2. 컨테이너의 모든 요소들은 순서를 갖는다. 각 컨테이너는 자신의 요소를 순회할 수 있도록 반복자를 제공한다.
3. STL 자체는 예외를 발생시키지 않는다.

* 컨테이너의 공통적인 동작


| 동작 | 효과 |
|---|:---:|
| `ContType c` |빈 컨테이너를 생성|
| `ConType c1(c2)` | 같은 타입의 컨테이너를 복사 |
| `ConType c(beg, end)` | 컨테이너를 생성하고 [beg, end)의 모든 원소들을 복사하여 초기화|
| `c.~ConType()` | 모든 원소들을 지우고 메모리 해제|
| `c.size()` | 현재 원소의 개수를 반환|
| `c.empty()` | 컨테이너가 비어 있는지 판단(size()==0보다 빠르다)|
| `c.max_size()` | 컨테이너가 가질 수 있는 최대 원소 갯수 반환|
| `== != < > <= >=` | c1, c2 비교 연산자|
| `c1 = c2` | c2의 모든 원소들을 c1에 할당|
| `c1.swap(c2)` | c1과 c2의 데이터를 교체|
| `swap(c1, c2)` | c1과 c2의 데이터를 교체(전역함수)|
| `c.begin()` | 첫 번째 원소의 반복자를 반환|
| `c.end()` | 마지막 원소의 뒷 위치를 가리키는 반복자 반환|
| `c.rbegin()` | 역방향의 첫 번째 원소를 가리키는 역방향 반복자 반환|
| `c.rend()` | 역방향의 마지막 원소의 뒷 위치를 가리키는 역방향 반복자 반환|
| `c.insert(pos, elem)` | 원소의 복사본을 삽입|
| `c.erase(beg, end)` | [beg, end)의 모든 원소들을 제거|
| `c.clear()` | 모든 원소들을 제거, 빈 컨테이너|
| `c.get_allocator()` | 컨테이너의 메모리 모델을 반환|

  - 다른 컨테이너 원소로 초기화
  ~~~cpp
  std::list<int> l;
  std::vector<float> c(l.begin(), l.end());
  ~~~
  - 배열의 원소로 초기화
  ~~~cpp
  int array[] = {2, 3, 17, 33, 45}
  std::set<int> c(array, array+sizeof(array)/sizeof(array[0]))
  ~~~
  - 표준 입력을 통하여 초기화
  ~~~cpp
  std::deque<int> c((std::istream_iterator<int>(std::cin)), (std::istream_iterator<int>()));
  ~~~
