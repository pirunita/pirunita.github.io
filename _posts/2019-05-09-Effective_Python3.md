---
layout: post
title:  "[Python] Effective Python(3) - iterate"
date:   2019-05-09 23:00:00 +0900
lang: ko
tags: Python
---
## range보다는 enumerate ##
&nbsp;&nbsp;내장 함수 range는 정수 집합을 순회(iterate)하는 루프, 문자열의 리스트 순회할 때 좋다. 하지만 리스트를 순회할 때 아이템의 인덱스를 알고 싶을 때 **enumerate**를 사용한다. enumerate는 지연 제너레이터로 iterator를 감싼다.
~~~python
for i, color in enumerate(color_list):
    print('%d: %s' %(i + 1, color))
~~~
enumerate로 시작 숫자를 지정(두 번째 파라미터, default는 0)하면 더욱 코드가 간결해 진다.
~~~python
for i, color in enumerate(color_list, i):
    print('%d: %s' %(i, color))
~~~


## iterator를 병렬로 처리하려면 zip을 사용 ##
&nbsp;&nbsp;리스트에서 표현식을 적용하여 파생 리스트를 얻을 수 있는 것은 저번 포스트에서 보았다.

~~~python
colors = ['red', 'green', 'blue']
letters = [len(n) for n in colors]
~~~
이 때 파생 리스트의 아이템과 소스 리스트의 아이템은 서로 인덱스로 연관되어 있다. 두 리스트를 병렬로 순회하려면 소스 리스트의 길이만큼 순회한다.

~~~python
longest_name = None
max_letters = 0
for i in range(len(colors)):
    count = letters[i]
    if count > max_letters:
        longest_name = colors[i]
        max_letters = count

print(longest_name)
~~~
하지만 루프문의 가독성이 떨어진다. colors와 letters를 인덱스로 접근하여 코드 읽기가 어려워지고 인덱스 i로 배열에 접근하는 동작이 두 번 일어난다. enumerate를 사용하면 문제점을 약간 개선할 수 있지만 완벽하진 않다.
~~~python
for i, color in enumerate(colors):
    count = letters[i]
    if count > max_letters:
        longest_name = color
        max_letters = count
~~~
<br>

이를 명확히 하는 내장 함수 zip을 이용하여 지연 제너레이터로 iterator 두 개 이상을 감싼다.
~~~python
for color, count in zip(colors, letters):
    if count > max_letters:
        longest_name = color
        max_letters = count
~~~
zip에는 두 가지 문제가 있다.
1. 파이썬2에서 zip은 제너레이터가 아니다.
2. 입력 iterator들의 길이가 다르면 zip이 이상하게 동작한다. zip은 감싼 iterator가 끝날 때 까지 tuple을 계속 넘겨준다.

## try/except/else/finally를 잘 이용한다. ##

### finally 블록 ###
&nbsp;&nbsp;예외가 발생해도 코드를 실행하고 싶을 때 try/finally를 사용한다.

~~~python
handle = open('/tmp/random_data.txt') # IOError가 일어날 수 있음.
try:
    data = handle.read() # UnicodeDecodeError가 일어날 수 있음.
finally:
    handle.close() # try: 이후 항상 실행
~~~
### else 블록 ###
&nbsp;&nbsp;코드에서 어떤 예외를 처리하고 어떤 예외를 전달할 지 명확히 하기 위해 try/except/else를 사용한다. try 블록이 예외를 일으키지 않으면 else 블록이 실행된다.
~~~python
def load_json_key(data, key):
    try:
        result_dict = json.loads(data) # ValueError가 일어날 수 있음.
    except ValueError as e:
        raise keyError from e
    else:
        return result_dict[key] # keyError가 일어날 수 있음.
~~~
만약 try에서 ValueError가 일어나면 except에서 발견되어 처리되고 디코딩이 성공하면 else 블록에서 키를 찾는다.
### 모두 사용 ###
복합문 하나로 모든 것을 처리하고 싶다면 try/except/else/finally를 모두 사용한다.
* try: 파일을 읽고 처리
* except: try에서 일어난 예외 처리
* else: 파일을 즉석에서 업데이트하고 관련된 예외를 전달
* finally: 파일 핸들을 정리

~~~python
UNDEFINED = object()

def divide_json(path):
    handle = open(path, 'r+')       #IOError가 일어날 수 있음
    try:
        data = handle.read()        # UnicodeDecodeError가 일어날 수 있음.
        op = json.loads(data)       # ValueError가 일어날 수 있음
        value = (
            op['numerator'] / op['denominator']
                                    # ZeroDivisionError가 일어날 수 있음.
        )
    except ZeroDivisionError as e:
        return UNDEFINED
    else:
        op['result'] = value
        result = json.dumps(op)
        handle.seek(0)
        handle.write(result)        # IOError가 일어날 수 있음
        return value
    finally:
        handle.close()
~~~

<br>
<hr>
브렛 슬라킨, ⌜파이썬 코딩의 기술⌟, 김형철, 길벗(2017)