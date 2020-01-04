---
layout: post
title:  "[Python] Effective Python(9) - Class & inheritance(3)"
date:   2019-05-27 13:00:00 +0900
lang: ko
tags: Python
---
<hr>

## 객체를 범용으로 생성하기 위해 @classmethod 이용하기 ##
&nbsp;&nbsp;다형성(Polymorphism)은 계층 구조에 속한 여러 클래스가 자체적인 메서드를 독립적으로 구현한다. 이를 통해 여러 클래스가 **같은 인터페이스, 추상 기반 클래스**를 가지면서 다른 기능을 제공할 수 있다.

~~~python

# 입력 데이터 클래스
class InputData(object):
    def read(self):
        raise NotImplementedError

# InputData의 서브 클래스
class PathInputData(InputData):
    def __init__(self, path):
        super().__init__()
        self.path = path
    
    def read(self):
        return open(self.path).read()

# 입력 데이터를 처리하는 작업 클래스
class Worker(object):
    def __init__(self, input_data):
        self.input_data = input_data
        self.result = None
    
    def map(self):
        raise NotImplementedError
    
    def reduce(self, other):
        raise NotImplementedError

# Worker의 서브클래스
class LineCountWorker(Worker):
    def map(self):
        data = self.input_data.read()
        self.result = data.count('\n')
    
    def reduce(self, other):
        self.result += other.result
~~~

하지만 위의 클래스들을 연결시키기 위해 헬퍼 함수로 직접 객체를 만들 수 있지만 범용적으로 사용하기 위한 특별한 셍성자를 만들기 위해 **@classmethod 를 이용한다.

~~~python
class InputData(object):
    def read(self):
        raise NotImplementedError
    
    @classmethod
    def generate_inputs(cls, config):
        raise NotImplementedError

class PathInputData(InputData):
    def read(self):
        return open(self.path).read()
    
    @classmethod
    def generate_inputs(cls, config):
        date_dir = config['data_dir']
        for name in os.listdir(data_dir):
            yield cls(os.path.join(data_dir, name))
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