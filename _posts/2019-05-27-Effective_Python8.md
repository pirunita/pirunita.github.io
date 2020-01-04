---
layout: post
title:  "[Python] Effective Python(8) - Class & inheritance(2)"
date:   2019-05-27 13:00:00 +0900
lang: ko
tags: Python
---
<hr>

## 인터페이스가 간단하면 클래스 대신 함수를 받는다. ##
&nbsp;&nbsp;파이썬 내장 API의 상당수는 함술르 넘겨서 동작을 사용자화 하는 기능을 후크(hook)라고 하며 API는 후크를 이용해 코드 실행 중에 호출한다.

파이썬의 후크 중 상당수는 인수와 반환 값을 잘 정의해 놓은 상태가 없는 함수이며 추상 클래스로 정의되지 않았다. 함수가 후크로 동작하는 것은 파이썬이 **first-class function**을 갖췄으며 <u>언어에서 함수와 메서드를 다른 값처럼 전달하고 참조할 수 있다.</u>

다음 예는 defaultdict 클래스의 동작을 사용자화 한다. 이 자료구조는 찾을 수 없는 키에 접근할 때 호출될 함수를 받는다. 키를 찾을 수 없을 때 로그를 남기고 기본값으로 0을 리턴하는 후크를 정의해 보자.
~~~python
def log_missing():
    print('Key added')
    return 0


current = {'green': 12, 'blue': 3}
increments = [
    ('red', 5), ('blue', 17), ('orange', 9)]
result = defaultdict(log_missing, current)
print('Before', dict(result))
for key, amount in increments:
    result[key] += amount
print('After', dict(result))

>>>
Before: {'green': 12, 'blue': 3}
Key added
Key added
After: {'orange': 9, 'green': 12, 'blue': 20, 'red': 5}
~~~
log_missing과 같은 함수를 넘기면 결정 동작과 부작용을 분리하여 API를 쉽게 구축하고 테스트할 수 있다. 또한 **상태 보존 클로저**를 기본값 후크로 사용하는 헬퍼 함수를 작성할 수도 있다.
~~~python
def increment_with_report(current, increments):
    added_count = 0

    def missing():
        nonlocal added_count # 상태 보존 클로저
        added_count += 1
        return 0
    
    result = defaultdict(missing, current)
    for key, amount in increments:
        result[key] += amount
    
    return result, added_count
~~~
increment_with_report 함수를 실행하면 튜플의 요소로 기대한 개수 2를 얻는다. 클로저 안에 상태를 숨기면 기능을 추가하기도 쉽다.
~~~python
result, count = increment_with_report(current, increments)
assert count == 2
~~~
하지만 상태 보존 후크용으로 클로저를 정의할 때 문제는 상태가 없는 함수의 예제보다 이해하기 어렵다. 따라서 보존할 상태를 캡슐화 하는 작은 클래스로 정의한다.
~~~python
class CountMissing(object):
    def __init__(self):
        self.added = 0
    
    def missing(self):
        self.added += 1
        return 0
~~~
파이썬은 first-class function이므로 객체로 CountMissing.missing 메서드를 직접 참조해서 defaultdict의 기본값 후크로 넘길 수 있다.
~~~python
count = CountMissing()
result = defaultdict(counter.missing, current)

for key, amount in increments:
    result[key] += amount
assert counter.added == 2
~~~
헬퍼 클래스로 상태 보존 클로저의 동작을 제공하는 방법이 앞서 함수를 사용한 방법보다 명확하지만 **CountMissing**클래스 자체만으로 용도가 무엇인지 바로 이해하기 어렵다. defaultdict와 연계해서 사용한 예를 보기 전까지는 알기 어렵기 때문에 **__ call __** 이라는 메서드를 정의해서 이를 명확하게 할 수 있다.

*__ call __메서드는 객체를 함수처럼 호출할 수 있게 해 주며 내장 함수 callable이 이러한 인스턴스에 대해 True를 반환하게 한다.*

~~~python
class BetterCountMissing(object):
    def __init__(self):
        self.added = 0

    def __call__(self):
        self.added += 1
        return 0


counter = BetterCountMissing()
counter()
assert callable(counter)

# BetterCountMissing 인스턴스를 defaultdict의 기본값 후크로 사용
counter = BetterCountMissing()
result = defaultdict(counter, current)
for key, amount in increments:
    result[key] += amount
assert counter.added == 2
~~~

__ call __ 메서드는 함수 인수를 사용하기 적합한 위치에 클래스의 인스턴스를 사용할 수 있다는 것을 알려주며 이 클래스의 주요 동작 진입점을 안내하고 클래스의 목적이 **상태 보존 클로저**로 동작할 것이라는 힌트를 제공한다. 또한 defaultdict는 여전히 기본값 후크용 함수만 필요하므로 클로저와 분리되어 있다.

<br>
<hr>
브렛 슬라킨, ⌜파이썬 코딩의 기술⌟, 김형철, 길벗(2017)