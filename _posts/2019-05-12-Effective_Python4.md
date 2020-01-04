---
layout: post
title:  "[Python] Effective Python(4) - Function(1)"
date:   2019-05-12 14:00:00 +0900
lang: ko
tags: Python
---
<hr>

## return None보다는 예외로 처리 ##
&nbsp;&nbsp;어떤 숫자를 다른 숫자로 나누는 함수에서 0으로 나누는 경우 None을 반환하는 것이 자연스러워 보인다.

~~~python
def divide(a, b):
    try:
        return a / b
    except ZeroDivisionError:
        return None

result = divide(x, y)
if result is None:
    print('Invalid inputs')
~~~

하지만 분자가 0인 경우 반환값은 0(분모는 0이 아님)이 된다. 그러면 결과 평가할 때 혹은 실수로 False에 해당하는 값을 검사할 시 문제가 된다. 이는 **None에 특별한 의미를 부여할 때** 오류가 발생한다.
~~~python
x, y = 0, 5
result = divide(x, y)
if not result:
    print('Invalid inputs') # False인 지 검사
~~~
<br>

이러한 오류들을 줄이기 위해 두 가지 방법이 있는데 여기서는 None을 반환하지 않는 방법만 알아본다. 호출쪽에 예외를 일으켜서 호출하는 쪽에서 그 예외를 처리하도록 한다.
~~~python
def divide(a, b):
    try:
        return a / b
    except ZeroDivisionError as e:
        raise ValueError('Inavlid inputs') from e
~~~
호출하는 쪽에서는 반환 값을 조건식으로 검사하지 않고 예외를 일으키지 않았다면 반환 값은 문제가 없다.
~~~python
x, y = 5, 2
try:
    result = divide(x, y)
except ValueError:
    print('Invalid inputs')
else:
    print('Result is %.1f' % result)
~~~

## 클로저가 변수 스코프와 상호 작용하는 방법 ##
&nbsp;&nbsp;리스트를 정렬할 때 특정 리스트의 값들을 먼저 오도록 우선순위를 매길 때 **리스트의 sort 메서드에 헬퍼 함수를 key 인수로 넘기는 것**이다. 이 때 헬퍼의 반환 값은 리스트의 각 값을 정렬하는 값으로 사용된다.
~~~python
def sort_priority(values, group):
    def helper(x):
        if x in group:
            return (0, x)
        return (1, x)
    values.sort(key=helper)

numbers = [8, 3, 1, 2, 5, 4, 7, 6]
group = [2, 3, 5, 7]
sort_priority(numbers, group)
print(numbers)

>>>
[2, 3, 5, 7, 1, 4, 6, 8]
~~~
* 파이썬은 **자신이 정의된 스코프에 있는 변수를 참조하는 함수**인 **클로저(closure)를 지원한다.
* 함수는 파이썬에서 first-class object이며, 함수를 직접 참조, 변수에 할당, 다른 함수의 인수 전달, 표현식과 if문 등에서 비교할 수 있기 때문에 클로저 함수를 key 인수로 받을 수 있다.
* 파이썬에는 튜플을 비교하는 특정한 규칙이 존재한다. Index 0으로 Item을 비교하고 Index 1, Index 2 순으로 진행한다.

다음과 같이 코드를 수정한다.
~~~python
def sort_priority2(numbers, group):
    found = False
    def helper(x):
        if x in group:
            found = True
            return (0, x)
        return (1, x)
    numbers.sort(key=helper)
    return found

found = sort_priority2(numbers, group)
print('Found: ', found)
print(numbers)

>>>
Found: False
[2, 3, 5, 7, 1, 4, 6, 8]
~~~
정렬된 결과는 올바르지만 스코프 탐색 순서 결과 False를 반환한다. 표현식에서 변수를 참조할 때 파이썬 인터프리터는 참조를 해결하기 위해 다음과 같은 순서로 스코프를 탐색한다.
1. 현재 함수의 스코프
2. 감싸고 있는 스코프
3. 코드를 포함한 모듈의 스코프(전역 스코프)
4. 내장 스코프

따라서 역순일 경우 NameError 예외가 일어난다. 따라서 found 변수는 helper 클로저에서 True로 할당되며 이는 sort_priority2에서 할당이 일어난 것이 아니고 helper 클로저 안에서 found라는 새로운 변수로 정의된 것이다.

하지만 클로저에서 데이터를 얻어오는 특별한 문법이 있다. **nonlocal**문은 특정 변수 이름에 할당할 때 스코프 탐색이 일어나야 함을 나타낸다.

~~~python
def sort_priority3(numbers, group):
    found = False
    def helper(x):
        nonlocal found
        if x in group:
            found = True
            return(0, x)
        return(1, x)
    number.sort(key=helper)
    return found
~~~
**nonlocal**문은 클로저에서 데이터를 다른 스코프에 할당하는 시점을 알아보기 쉽게 해준다. 변수 할당이 모듈 스코프에 직접 들어가는 global문을 보완한다. 하지만 *할당이 멀리 떨어진 긴 함수*는 자제하도록 한다. 따라서 nonlocal을 사용할 때 복잡해지기 시작하면 helper 클래스로 상태를 감싼다.

__call__이라는 특별한 메서드는 뒤에서 설명한다.
~~~python
class Sorter(object):
    def __init__(self, group):
        self.group = group
        self.found = False
    
    def __call__(self, x):
        if x in self.group:
            self.found = True
            return(0, x)
        return(1, x)

sorter = Sorter(group)
numbers.sort(key=sorter)
~~~
<br>
<hr>
브렛 슬라킨, ⌜파이썬 코딩의 기술⌟, 김형철, 길벗(2017)