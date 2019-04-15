---
layout: post
title:  "Binary Search"
date:   2019-04-15 23:00:00 +0900
lang: ko
tags: ProblemSolving
---

*1일 1커밋을 목표로 했지만 시간이 없으니 예전에 정리해 둔 binary search를 올리려고 한다.*<br>
*원래 JAVA로 Problem Solving을 했지만 진짜 코드가 길고 귀찮아서 역시 언어를 바꿔야겠다는 생각에*<br>
*빠르게 C++ 기초를 읽고 hackerrank로 PS를 시작하게 되는데...*
<br>
<br>


<a href="http://www.cplusplus.com/reference/algorithm/lower_bound/">http://www.cplusplus.com/reference/algorithm/lower_bound/</a>
<br>
&nbsp;&nbsp;Binary Search(이분 탐색)은 정렬되어 있는 배열에서 우리가 찾고 싶은 값을 빠르게 찾기 위한 탐색 방법이다. 이분 탐색인 것은 탐색 범위를 계속 반으로 줄여나가며 찾기 때문이다.<br>
&nbsp;&nbsp;이 때 우리가 찾고싶은 값들과 관련해서 두 가지의 위치로 나타낼 수 있다. **lower bound**, **upper bound**이며 STL에도 잘 구현되어 있다.
* Lower bound : Lower bound는 우리가 찾고 싶은 값 **이상(포함)**이 처음으로 나오는 위치이다.
* Upper bound : Upper bound는 우리가 찾고 싶은 값 **초과**가 처음으로 나오는 위치이다.<br>
예시를 한 번 보자
<br>
~~~
[1, 2, 3, 3, 4, 5]
~~~
이 때 우리가 찾고 싶은 값이 3이라면 **Lower bound**는 처음 등장하는 3 이상인 값인 **3번째 위치**의 3이고<br>
**Upper bound**는 처음 등장하는 3보다 큰 값인 **5번째 위치**의 4이다.
<br><br>
개념을 알았으니 구현은 생략하고 STL을 사용 해 본다.
<br><br>

#### Example ####
(다음은 cpp reference document에 있던 예제이다.)
~~~cpp
// lower_bound/upper_bound example
#include <iostream>     // std::cout
#include <algorithm>    // std::lower_bound, std::upper_bound, std::sort
#include <vector>       // std::vector

int main () {
  int myints[] = {10,20,30,30,20,10,10,20};
  std::vector<int> v(myints,myints+8);
  // 10 20 30 30 20 10 10 20

  std::sort (v.begin(), v.end());
  // 10 10 10 20 20 20 30 30

  std::vector<int>::iterator low,up;
  low=std::lower_bound (v.begin(), v.end(), 20);
  up= std::upper_bound (v.begin(), v.end(), 20);

  std::cout << "lower_bound at position " << (low- v.begin()) << '\n';
  std::cout << "upper_bound at position " << (up - v.begin()) << '\n';

  return 0;
}
~~~
Output:
~~~
lower_bound at position 3
upper_bound at position 6
~~~
low는 20 이상이 처음 나오는 위치이기 때문에 4번째 즉 v[3]에 위치한 20이고,<br>
up 은 20 초과가 처음 나오는 위치이기 때문에 7번째 즉 v[6]에 위치한 30이다.<br>