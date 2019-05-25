---
layout: post
title:  "[Python] Effective Python(6) - Function(3)"
date:   2019-05-25 10:00:00 +0900
lang: ko
tags: Python
---
## 가변 위치 인수로 깔끔하게 보이게 한다. ##
&nbsp;&nbsp;선택적인 위치 인수(*args)를 받게 만들면 함수 호출을 더 명확하게 할 수 있다. 다음과 같이 디버그 정보를 로그로 남기는 예시를 보자.
~~~python
def log(message, values):
    if not values:
        print(message)
    else:
        value_str = ', '.join(str(x) for x in values)
        print('%s: %s' % (merssage, values_str))

log('My numbers are ', [1, 2])
log('Hi there', [])

>>>
My numbers are 1, 2
Hi there
~~~
로그로 남길 값이 없을 때 빈 리스트를 넘겨야 한다는 것이 문제이다. 따라서 두 번째 인수에 * 기호를 마지막 위치 파라미터 이름 앞에 붙여서 인수를 남긴다.

~~~python
def log(message, *values):
    if not values:
        print(message)
    else:
        value_str = ', '.join(str(x) for x in values)
        print('%s: %s' % (merssage, values_str))

log('My numbers are ', 1, 2)
log('Hi there')

>>>
My numbers are 1, 2
Hi there
~~~
리스트를 log같은 가변 인수 함수를 호출하는데 사용하고 싶을 때 __* 연산자__ 를 사용하며 파이썬은 리스트에 들어있는 아이템을 위치 인수로 전달한다.

가변 개수의 위치 인수를 받는 방법에는 두 가지 문제가 있다.
1. **가변 인수가 함수에 전달되기 전에 항상 튜플로 변환된다.** 함수를 호출하는 쪽에서 제너레이터에 *연산자를 쓰면 제너레이트가 모두 소진될 때 까지 순회된다. 이 때 결과로 만들어지는 튜플은 제너레이터로부터 생성된 모든 값을 담으므로 메모리를 많이 차지하게 된다.

~~~python
def my_generator():
    for i in range(10):
        yield i


def my_func(*args):
    print(args)


it = my_generator()
my_func(*it)

>>>
(0, 1, 2, 3, 4, 5, 6, 7, 8, 9)
~~~

*args를 받는 함수는 인수 리스트에 있는 입력의 수가 많지 않을 때 가장 좋은 방법이다. *args를 받는 함수는 많은 literal값, 변수 이름을 넘길 때 이상적이다.

2. **추후에 호출 코드를 모두 변경하지 않고서 새 위치 인수를 추가할 수 없다.**

~~~python
def log(sequence, message, *values):
    if not values:
        print('%s %s' % (sequence, message))
    else:
        values_str = ', '.join(str(x) for x in values)
        print('&s: &s: &s' & (sequence, message, values_str))


log(1, 'Favorites', 7, 33)
log('Favorites', 7, 33)

>>>
1: Favorites: 7, 33
Favorites: 7: 33
~~~

이 코드는 두 번째 호출에서 sequence를 받지 못했기 때문에 7을 massage 파라미터로 사용한다. 이를 완전히 없애려면 *args를 받는 함수를 확장할 때 **키워드 전용 인수**를 사용해야 한다.

## 키워드 인수로 선택적인 동작을 제공한다. ##
&nbsp;&nbsp;파이썬에서도 함수를 호출할 때 인수를 위치로 전달할 수 있다. 또한 **파이썬 함수의 위치 인수를 모두 키워드로 전달할 수 있다.** 필요한 위치 인수를 모두 저장한다면 키워드 인수로 전달할 수 있으며 섞어서도 가능하다.
~~~python
remainder(20, 7)
remainder(20, divisor=7)
remainder(number=20, divisor=7)
remainder(division, number=20)
~~~
하지만 위치 인수는 키워드 인수 앞에 지정해야 한다.
~~~python
remainder(number=20, 7)

>>>
SyntaxError: non-keyword arg after keyword arg
~~~
또한 각 인수는 한 번만 지정 가능하다.
~~~python
remainder(20, number=7)

>>>
TypeError: remainder() got multiple values for argument 'number'
~~~
키워드 인수의 유연성은 세 가지 이점이 있다.
1. 코드를 처음 보는 사람이 함수 호출을 더 명확하게 이해할 수 있다.
2. 함수르 정의할 때 기본값을 설정할 수 있다.
~~~python
def flow_rate(weight_diff, time_diff, period=1):
    return (weight_diff / time_diff) * period
~~~
3. 기존의 호출 코드와 호환성을 유지하면서도 함수의 파라미터를 확장할 수 있다.
~~~python
def flow_rate(weight_diff, time_diff, period=1, units_per_kg=2.2):
    return (weight_diff / unites_per_kg / time_diff) * period
~~~
이 방법의 유일한 문제는 period, units_per_kg를 위치 인수로도 넘길 수 있다. 따라서 항상 키워드 이름으로 선택적인 인수를 지정하고 위치 인수로는 넘기지 않는 것이 좋다.

## 동적 기본 인수를 지정하려면 None과 docstring을 사용한다. ##
&nbsp;&nbsp; 키워드 인수의 기본값으로 비정적(non-static) 타입을 사용해야 할 때가 있다. 다음 이벤트 발생 시각 로깅 메시지 예시를 본다.
~~~python
def log(message, when=datetime.now()):
    print('%s: %s' % (when, message))


log('Hi there')
sleep(0.1)
log('Hi there')

>>>
2014-11-15 21:10:10.371432: Hi there
2014-11-15 21:10:10.371432: Hi there
~~~
이 때 datetime.now는 함수르 정의할 때 한 번만 실행되므로 타임스탬프가 동일하게 출력된다. 기본 인수의 값은 모듈이 로드될 때 한 번만 평가된다. 따라서 호출될 때 기본 인수를 평가하기 위해 기본값을 None으로 설정하고 doctring(문서화 문자열)로 실제 동작을 문서화한다.
~~~python
def log(message, when=None):
    """Log a message with a timestamp.
    Args:
        message: Message to print.
        when: datetime of when the message occured.
            Defaults to the present time.
    """
    when = datetime.now() if when if None else when
    print('%s: %s' % (when, message))

>>>
2014-11-15 21:10:10.472303: Hi there
2014-11-15 21:10:10.573395: Hi there
~~~

<br>
<hr>
브렛 슬라킨, ⌜파이썬 코딩의 기술⌟, 김형철, 길벗(2017)