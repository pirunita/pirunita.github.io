---
layout: post
title:  "[Python] Effective Python(1) - Style Guide"
date:   2019-05-09 09:00:00 +0900
lang: ko
tags: Python
---

&nbsp;&nbsp;_현재 내가 대부분 진행하는 프로젝트는 python으로 짜여 있다. 하지만 python을 제대로 공부한 적이 없으며 pythonic하게 코드를 짜는 방법에 대해서 잘 알지 못하기에 이번에 공부하고자 한다._

## PEP 8 스타일 가이드 ##
&nbsp;&nbsp;파이썬 개선 제안서(Python Enhancement Proposal) #8 즉, PEP 8은 파이썬 코드를 어떻게 구성할 지 알려주는 스타일 가이드다. 이 때 반드시 따라야 하는 규칙이 있다.

### 화이트 스페이스 ###
* 탭이 아닌 스페이스로 들여쓴다.
* 문법적으로 의미가 있는 들여쓰기는 스페이스 4개를 사용한다.
* 한 줄의 문자 길이가 79자 이하여야 하며 표현식이 길어서 다른 줄로 이어지면 일반적인 들여쓰기에서 스페이스 네 개를 추가로 쓴다.
* 파일에서 함수와 클래스는 빈 줄 두개로 구분해야 한다.
* 클래스에서 메서드는 빈 줄 하나로 구분한다.
* 리스트 인덱스, 함수 호출, 키워드 인수 할당에는 스페이스를 사용하지 않는다.
* 변수 할당 앞 뒤에 스페이스를 하나만 사용한다.

### 명명 표기 ###
* 함수, 변수, 속성은 lowercase_underscore 형식을 따른다.
* protected 인스턴스 속성은 _leading_underscore 형식을 따른다.
* private 인스턴스 속성은 __double_leading_underscore 형식을 따른다.
* 클래스와 예외는 CapitalizedWord 형식을 따른다.
* 모듈 수준 상수는 ALL_CAPS 형식을 따른다.
* 클래스의 인스턴스 메서드에서 첫 번째 파라미터(해당 객체를 참조)의 이름을 self로 지정한다.
* 클래스 메서드에서 첫 번째 파라미터(해당 클래스를 참조)의 이름을 cls로 지정한다.

### 표현식과 문장 ###
* 길이를 확인하는 (if len(somelist) == 0)이 아닌 if not some list를 사용하고, 빈 값은 암시적으로 False가 된다고 가정한다.
* 비어있지 않은 값에도 위와 같다. 값이 비어있지 않으면 if omselist 문이 암시적으로 True가 된다.
* 한줄로 된 if, for, while, except을 쓰지 않는다.
* 파일 맨 위에 import문을 놓는다.
* module을 import 할 때 항상 모듈의 절대 이름을 사용한다.
* Import는 '표준 라이브러리 모듈, 서드파티 모듈, 자신이 만든 모듈' 섹션 순으로 구분한다. 각각의 하위 섹션에서는 알파벳 순서로 import 한다.

## 복잡한 표현식 대신 헬퍼 함수를 작성한다. ##
&nbsp;&nbsp;파이썬의 간결한 문법을 이용하면 짧은 로직을 표현식 한 술로 작성할 수 있지만 읽기 어려워지는 경우가 있다.
다음은 URL에서 쿼리 문자열을 디코딩하는 코드이다.
~~~python
from urllib.parse import parse_qs
my_values = parse_qs('red=5&blue=0&green=', keep_blank_values=True)
print(repr(my_values))


>>>
{'red': ['5'], 'green': [''], 'blue': ['0']}
~~~

Dictionary의 get 메서드를 사용하여 각 상황에 따라 다른 값을 반환한다.
~~~python
print('red: ', my_values.get('red'))
print('green: ', my_values.get('green'))
print('purple: ', my_values.get('purple'))


>>>
red: ['5']
green: ['']
purple: None
~~~
이 때 파라미터가 없거나 비어 있으면 0을 할당하게 한다.
~~~python
red = my_values.get('red', [''])[0] or 0
green = my_values.get('green', [''])[0] or 0
purple = my_values.get('purple', [''])[0] or 0
print('red: %r', %red)
print('green: %r', %green)
print('purple: %r', %purple)


>>>
red: '5'
green: 0
purple: 0
~~~
if/else 조건식(삼항 표현식)을 이용하여 명확하게 표현한다.
~~~python
red = my_values.get('red', [''])
red = int(red[0]) if red[0] else 0
~~~
이 로직을 반복해서 사용한다면 헬퍼 함수를 만든다.
~~~python
def get_first_int(values, key, default=0):
    found = values.get(key, [''])
    if found[0]:
        found = int(found[0])
    else
        found = default
    return found

green = get_first_int(my_values, 'green')
~~~