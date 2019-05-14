---
layout: post
title:  "[Python] Effective Python(5) - Function(2)"
date:   2019-05-14 20:00:00 +0900
lang: ko
tags: Python
---
## List를 반환하는 대신 generator를 고려한다. ##
&nbsp;&nbsp;어떤 결과를 생성하는 함수에서 리스트를 반환하는 것이 가장 간단한 방법이다. 다음은 문자열에 있는 모든 단어의 인덱스를 출력하는 것이다.
~~~python
def index_words(text):
    result = []
    if text:
        result.append(0)
    for index, letter in enumerate(text):
        if letter == ' ':
            result.append(index + 1)
    return result
~~~
하지만 위와 같은 코드는 문제점이 있다.
1. 코드가 복잡하다. 메서드 호출(result.append)가 많다.
2. 반환하기 전에 모든 결과를 리스트에 저장해야 한다. 이는 프로그램 실행 중에 입력이 많으면 메모리가 고갈 될 여지가 있다.

1번 문제를 해결하기 위해서는 **제너레이터**를 사용한다. 제너레이터는 **yield 표현식**을 사용하는 함수이며 이는 실제로 실행되지 않고 바로 **이터레이터(iterator)**를 반환하고 내장함수 next를 호출할 때마다 이터레이터는 제너레이터가 다음 yield 표현식으로 진행하게하고 제너레이터에서 yield에 전달한 값을 이터레이터가 호출하는 쪽에 반환한다.

~~~python
def index_words_iter(text):
    if text:
        yield 0
    for index, letter in enumerate(text):
        if letter == ' ':
            yield index + 1

result = list(index_words_iter(address))
~~~
이렇게 되면 결과 리스트와 연동하는 부분이 없어지고 결과는 yield 표현식으로 전달(반환되는 이터레이터)되어 내장 함수 list에 전달할 수 있다.

이제 2번 문제를 해결하기 위해 제너레이터로 작성한 버전에서 다양한 길이의 입력을 대응하도록 한다. 다음은 파일에서 입력을 한 번에 한 줄씩 읽어서 한 번에 한 단어씩 출력을 내는 제너레이터이다. 함수가 동작할 때 사용하는 메모리는 입력 한 줄의 최대 길이까지다.

~~~python
def index_file(handle):
    offset = 0
    for line in handle:
        if line:
            yield offeset
        for letter in line:
            offset += 1
            if letter == ' ':
                yield offset

with open('/tmp/address.txt', 'r') as f:
    it = index_file(f)
    results = islice(it, 0, 3)
    print(list(results))
~~~
주의할 점은 제너레이터를 정의할 때 반환되는 이터레이터에 상태가 있고 재사용할 수 없다는 것을 호출 쪽에 알아둔다.

## 인수를 순회할 때는 방어적으로 한다. ##
&nbsp;&nbsp;파라미터로 객체의 리스트를 받는 함수에서 리스트를 여러 번 순회할 때가 있다. 다음 예를 살펴 본다.

~~~python
# 정규화 함수
def normalize(numbers):
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result

visits = [15, 35, 80]
percentages = normalize(visits)
print(percentages)

>>>
[11.538461538461538, 26.923076923076923, 61.53846153846154]
~~~
이를 위에서 배운 제너레이터로 정의 해 보자.
~~~python
def read_visits(data_path):
    with open(data_path) as f:
        for line in f:
            yield int(line)

it = read_visits('/tmp/my_numbers.txt')
percentages = normalize(it)
print(percentages)

>>>
[]
~~~
이 때 제너레이터의 반환 값에 normalize를 호출하면 아무 결과가 생성되지 않는데 이는 이터레이터가 결과를 한 번만 생성하기 때문이다.
~~~python
it = read_visits('/tmp/my_numbers.txt')
print(list(it))
print(list(it))

>>>
[15, 35, 80]
[]
~~~
이미 소진한 이터레이터를 순회해도 오류가 일어나지 않는다. for, list 생성자, 파이썬 표준 라이브러리의 많은 함수는 StopIteration 예외가 일어날 거라고 기대하지만, 이미 소진한 이터레이터의 차이를 알려주지 않는다.

따라서 이를 해결 하기 위해 *입력 이터레이터를 명시적으로 소진하고 전체 콘텐츠의 복사본을 리스트에 저장한다.* 그러면 필요한 데이터만큼 순회한다.

~~~python
def normalize_copy(numbers):
    numbers = list(numbers)     # 이터레이터 복사
    total = sum(numbers)
    result = []
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result
~~~
하지만 이는 리스트를 복사할 때 크기가 크면 메모리 고갈이 일어날 수 있다. 따라서 *호출 될 때마다 새 이터레이터를 반환하는 함수를 받게 만든다.*
~~~python
def normalize_func(get_iter):
    total = sum(get_iter())     # 새 이터레이터
    result = []
    for value in get_iter():    # 새 이터레이터
        percent = 100 * value / total
        result.append(percent)
    return result

percentages = normalize_func(lambda: real_visits(path))
~~~
코드가 잘 동작하지만 람다 함수를 넘겨주는 방법은 깔끔하지 않다. 따라서 **이터레이터 프로토콜**을 구현한 새 컨테이너 클래스를 제공한다.

**iterator protocol**은 for루프와 관련 표현식이 컨테이너 타입의 콘텐츠를 탐색하는 방법을 나타낸다.

예를 들어 파이썬은 *for x in foo* 와 같은 문장을 만나면 실제로는 *iter(foo)*를 호출한다. 내장 함수 **iter**는 특별한 메서드인 **foo.__ iter__**를 호출하며 __ iter __ 메서드는 이터레이터 객체를 반환해야 한다. for루프는 이터레이터를 모두 소진할 때 까지 이터레이터 객체에 내장 함수 next를 계속 호출한다.

따라서 클래스의 **__ iter __** 메서드를 제너레이터로 구현한다.

~~~python
class ReadVisits(object):
    def __init__(self, data_path):
        self.data_path = data_path

    
    def __iter__(self):
        with open(self.data_path) as f:
            for line in f:
                yield int(line)

visits = ReadVisits(path)
percentages = normalize(visits)
print(percentages)
~~~
새로 정의한 컨테이너 타입은 원래의 함수에 수정을 가하지 않고 동작한다. 이는 normalize의 sum 메서드가 새 이터레이터 객체를 할당하기 위해 ReadVisits.__iter__를 호출하며 normalize 내부에서 for 루프도 두 번째 이터레이터 객체를 할당할 때 __ iter __를 호출한다. 두 이터레이터는 독립적으로 동작하므로 각각 순회하여 모든 데이터 값을 얻지만 *입력 데이터를 여러 번 읽는다* 는 단점이 있다.

이제 파라미터가 단순한 이터레이터가 아님을 보장하는 함수를 작성한다. 프로토콜은 내장 함수 iter에 이터레이터를 넘기면 이터레이터 자체가 반환된다. 하지만 iter에 컨테이너 타입을 넘기면 매번 새 이터레이터 객체가 반환되기 때문에 이 때는 입력값을 테스트해서 **TypeError**을 일으키게 만든다.

~~~python
def normalize_defensive(numbers):
    if iter(numbers) is iter(numbers):
        raise TypeError('Must supply a container')
    total = sum(numbers)
    result = []
    
    for value in numbers:
        percent = 100 * value / total
        result.append(percent)
    return result
~~~
normalize_defensive는 normalize_copy처럼 입력 이터레이터 전체를 복사하고 싶지 않지만 입력 데이터를 여러 번 순회해야 할 때 사용하면 좋다. 이 때 함수는 입력이 이터러블이어도 컨테이너가 아니면 예외를 일으킨다.


<br>
<hr>
브렛 슬라킨, ⌜파이썬 코딩의 기술⌟, 김형철, 길벗(2017)