---
layout: post
title:  "[Python] profiling"
date:   2019-05-03 13:00:00 +0900
lang: ko
tags: Python
---
## profiling으로 병목 지점 찾기 ##

* 코드의 속도상 병목과 RAM 사용 병목을 파악하는 방법
* CPU와 메모리 사용량을 프로파일링하는 방법
* 얼마나 자세하게 프로파일링을 해야 하는가
* 장시간 실행되는 어플리케이션을 프로파일링 하는 방법
* CPython의 내부 동작
* 성능을 튜닝하는 동안 코드의 올바름을 유지하는 방법
<hr>

&nbsp;&nbsp;프로파일링을 하게 되면 RAM이 사용될 때 몰리는 병목 지점을 찾을 수 있기 때문에, 프로파일링을 하면 속도를 크게 향상 시키고 자원을 적게 사용할 수 있다는 장점이 있다.

또한 CPU뿐만 아니라 GPU, RAM, 네트워크 대역폭, 디스크 I/O 등 측정 가능한 자원은 모두 프로파일링을 해 볼 수 있다.

### 2.1 효과적으로 프로파일링 하기 ###

프로파일링의 첫 번째 목표 : 시스템의 어느 부분이 느린 지, 병목 현상이 어디서 일어나는 지 알 수 있다.

(*주의: 프로파일링 시 평소보다 실행 속도가 느려지므로 검사할 부분만 따로 떼어내서 검사한다.*) 

프로파일링 방법은 다음과 같다.
* IPython의 %timeit 매직 명령어, time.time(), timing decorator
* cProfile: 어떤 함수가 가장 오래 걸리는 지 확인할 수 있는 내장 도구
* line_profiler: 선택한 함수를 한 줄씩 프로파일링
  - line_profiler을 통해 컴파일러를 사용할 부분을 결정할 수 있다.
* perf stat: CPU에서 실행된 명령의 수, CPU 캐시가 얼마나 효율적으로 활용 되었는 지 알 수 있다.(chap 6)
* heapy: 파이썬 메모리에 상주하는 모든 객체를 확인
  - 메모리 누수를 찾을 때 유용
  - 장시간 실행되는 시스템이라면 dowser를 사용한다.
* memory_profiler: 시간에 따른 RAM 사용 추이를 차트로 확인
  - RAM 사용량이 왜 높은 지 찾을 수 있다.
* CPython 내부에서 쓰이는 파이썬 바이트코드

### 2.2 시간을 측정하는 간단한 방법 - print와 데코레이터 ###
&nbsp;&nbsp;print문을 사용하는 경우는 코드를 더럽히지만 시간을 잠깐 조사하는 경우에 매우 유용하다.
(*작업이 끝나고 print문을 정리하지 않으면 표준 출력을 모두 잡아먹는다.*)

~~~python
import time

def print_time():
  start_time = time.time()
  output = func()
  end_time = time.time90
  secs = end_time - start_time
~~~

조금 더 깨끗한 방법으로 **데코레이터**가 있다. 여기선 시간을 측정하고 싶은 함수 위에 코드 한 줄을 추가할 것이다.

~~~python
from functools import wraps

def timefn(fn):
  @wraps(fn)
  def measure_time(*args, **kwargs):
    t1 = time.time()
    result = fn(*args, **kwargs)
    t2 = time.time()
    print("@timefn: {} took {} seconds".format(fn.__name__, t2- t1))
    return measure_time

@timefn
def calculate_z_serial_purepython(maxiter, zs, cs):
  ...
~~~

추가적인 프로파일링 정보는 코드를 느려지게 만들 수 있다.

이번에는 CPU를 많이 사용하는 함수의 실행 속도를 측정하는 다른 방법으로 timeit 모듈을 사용할 수 있다.

~~~
$ python -m timeit -n 5 -r 5 -s "import julia1" "julia1.calc_pure_python(desired_width=1000, max_iterations=300)"
~~~
여기서 **calc_pure_python**이 모듈 안에 정의되어 있으므로 -s를 사용하여 module을 임포트해야한다.

* -r 5: 반복횟수를 나타냄
* -n 5: 각 검사에서 평균을 구하여 전체 반복 중 가장 나은 값을 출력
* 만약 -n, -r 옵션을 주지 않고 timeit을 수행했다면 기본값으로 5번씩 10회 반복 수행

### 2.3 cProfile module 사용 ###
&nbsp;&nbsp;cProfile은 표준 라이브러리가 제공하는 3가지 프로파일러 중 하나이다.(hotshot, profile)

* tip: 프로파일링을하기 전에 프로파일링하려는 코드의 기대 속도에 대한 가설을 세우는 습관을 들이는 것이 좋다. 의심스러운 코드를 출력해서 메모하는 방식이 좋다. 
~~~
python -m cProfile -s cumulative julia1_nopil.py
~~~
* -s cumulative: 각 함수에서 소비한 누적 시간순으로 정렬되어 어떤 함수가 더 느린지 쉽게 확인 가능
~~~
python -m cProfile -o profile.stats julia1.py
~~~
* cProfile의 결과를 더 세밀하게 살펴보려면 통계 파일을 생성한 뒤 파이썬으로 분석할 수 있다. 이제 다음과 같이 파이썬에서 통계 파일을 불러들이면 앞서 살펴본 출력과 같은 결과를 확인할 수 있다.
~~~python
import pstats
p = pstats.Stats("profile.stats")
p.sort_stats("cumulative")
p.print_stats()
~~~

* 프로파일링 중인 함수를 추적하기 위해 해당 함수를 호출한 측(caller)의 정보를 출력한다.
~~~python
p.print_callers()
~~~
* 이를 통해 어떤 함수가 몇 번 호출했고 수행시간을 측정할 수 있다. 또한 다음 방법으로 해당 함수에서 호출하는 함수 목록도 확인할 수 있다.
~~~python
p.print_callees()
~~~

### 2.4 line_profiler로 한 줄씩 측정하기 ###
&nbsp;&nbsp;line_profiler은 파이썬 코드에서 CPU 병목 원인을 찾아주는 가장 강력한 도구일 것이다. *line_profiler는 개별 함수를 한 줄씩 프로파일링할 수 있으므로* cProfile로 어떤 함수를 line_profiler로 자세히 살펴볼 지에 대한 전체적인 방향을 정한다.

먼저 line_profiler를 설치한다.
~~~
pip install line_profiler
~~~

* @profiler: decorator, 선택한 함수를 표시하기 위해 사용
* kernprof.py: 코드를 실행하고 선택한 함수의 각 줄에 대한 CPU 시간 등의 통계를 기록
* -l: l옵션은 함수 단위가 아닌 **한 줄씩 프로파일링**
* -v: v옵션은 출력 결과를 다양하게 보여준다. -v를 주지 않으면 line_profiler 모듈을 이용한 분석에 사용할 수 있는 .lprof 결과를 받는다.

~~~
kernprof.py -l -v julia1_lineprofiler.py
~~~
이 내용에서 print만으로 걸린 측정 시간은 13초, cProfile은 19초, 하지만 line_profiler은 100초가 소요되지만 각 줄이 실행되는 데 걸린 시간을 확인할 수 있다.

* tip: while문에서 조건이 2개 이상이라면 이를 분리해서 if조건문으로 처리 해 준 후에 측정한다.

또한 파이썬은 다른 부분을 계산하지 않고 전체 결과를 확정할 수 있으면 전체를 확정지어 버리기 때문에 while문의 연산 순서를 바꿈으로써 유의미한 속도 개선을 기대할 수 있다.

### 2.5 memory_profiler로 메모리 사용량 진단하기 ###
&nbsp;&nbsp;메모리 사용량을 줄단위로 측정해주는 memory_profiler이다. 이를 통해서 다음을 알 수 있다.
* 이 함수를 더 효과적으로 작동하게 고쳐서 RAM을 덜 사용할 수 있는가
* 캐시를 두어 RAM을 조금 더 쓰는 대신 CPU 사이클을 아길 수 있을까?

memory_profiler는 line_profiler와 흡사하게 작동하지만 훨씬 느리다. psutil 패키지를 설치하면 memory_profiler를 더 빠르게 실행할 수 있다.
~~~
pip install memory_profiler
pip install psutil
~~~

memory_profiler은 느리기 때문에 최소한의 시간에 프로파일링을 마칠 수 있도록 코드의 일부만 떼어내 검사하는 편이 좋다.

~~~
python -m momry_profiler julia1_memoryprofiler.py
~~~

메모리 사용량의 변화를 시각화하는 방법은 시간에 따라 샘플링하고 그 결과를 그래프로 그려보는 것이다. memory_profiler에 포함된 mprof는 메모리 사용량을 샘플링한 후 시각화하며, 시간에 따라 샘플링하기 때문에 코드 실행시간에 큰 영향을 주지 않는다.

~~~
mprof run julia1_memoryprofiler.py
~~~
통계 파일을 먼저 생성하고 mprof plot 명령으로 시각화한다.

* range vs xrange?

### 2.6 최적화 중에 단위 테스트 하기 ###
&nbsp;&nbsp;단위 테스트를 하지 않으면 장기적으로 생산성을 떨어트린다. 단위 테스트와 더불어 coverage.py도 함께 고려한다. coverage.py는 코드의 어떤 부분이 검사되었고 검사되지 않은 부분은 어디인지 알 수 있다.

#### no-op @profiler 데코레이터 ####
&nbsp;&nbsp;line_profiler나 memory_profiler에서 @profile을 사용하면 단위테스트에서 NameError 예외가 발생한다. 단위 테스트 프레임워크는 @profile 데코레이터를 로컬 네임스페이스에 추가하지 않았기 때문에 **no-op decorator**를 이용하면 이 문제를 피할 수 있다. 검사하려는 코드 블록에 추가하고 검사가 끝나면 제거한다.

no-op를 이용하면 검사하려는 코드를 변경하지 않고 테스트를 실행할 수 있고, 프로파일링을 통한 최적화 작업 중에도 테스트를 돌려볼 수 있기 때문에 잘못된 최적화에 빠지는 일을 방지할 수 있다.

~~~python
# ex.py
import unittest

@profile
def some_fn(nbr):
    return nbr * 2

cass TestCase(unittest.TestCase):
    def test(self):
        result = some_fn(2)
        self.assertEquals(result, 4)
~~~

~~~
$ nosetests ex.py
E
================================================
ERROR: Failure: NameError (name 'profile' is not defined)
~~~

따라서 해결책은 ex.py의 첫 부분에 no-op 데코레이터를 추가하는 것이다.(프로파일링이 끝나고 제거하면 된다.)

* 단위 테스트를 위해 네임스페이스에 no-op @profiler 데코레이터 추가하기(line_profiler)

~~~python
# line_profiler용
if '__builtin__' not in dir() or not hasattr(__builtin__, 'profile'):
    def profile(func):
        def inner(*args, **kwargs):
            return func(*args, **kwargs)
        return inner
~~~

'__builtin __' 테스트는 nosetest 유무를 위한 것이고 hasattr 테스트는 네임스페이스에 @profile이 추가되었는 지 검사한다.

~~~
$ kernprof.py -v -l ex.py
Line #     Hits        Time Per %%HTMLit    % Time Line Contests
=====================================================================
  11                                              @profile
  12                                              def some_fn(nbr):
  13          1           3     3.0       100.0       return nbr * 2



$ nosetests ex.py
.
Ran 1 test in 0.000s
~~~

* 단위 테스트를 위해 네임스페이스에 no-op @profiler 데코레이터 추가하기(memory_profiler)

~~~python
# memory_profiler용
if 'profile' not in dir():
    def profile(func):
        def inner(*args, **kwargs):
            return func(*args, **kwargs)
        return inner
~~~

~~~
python -m memory_profiler ex.py
...
Line #      Mem usage     Increment     Line Contents
=====================================================================
    11      10.809 MiB    0.000 MiB     @profile
    12                                  def some_fn(nbr):
    13      10.809 MiB    0.000 MiB         return nbr * 2

$ nosestests ex.py
.
Ran 1 test in 0.000s
~~~



