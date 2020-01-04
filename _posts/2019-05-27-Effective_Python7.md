---
layout: post
title:  "[Python] Effective Python(7) - Class & inheritance(1)"
date:   2019-05-27 11:00:00 +0900
lang: ko
tags: Python
---
<hr>

## 딕셔너리와 튜플보다는 헬퍼 클래스로 관리 ##
&nbsp;&nbsp;파이썬의 딕셔너리 타입은 객체가 계속 지속될 때 동적인 내부 상태를 관리하는 용도로 좋다. 다음 예시처럼 학생별로 미리 정의된 속성을 사용하지 않고 딕셔너리에 이름을 저장하는 클래스를 정의할 수 있다.
~~~python
class SimpleGradebook(object):
    def __init__(self):
        self._grades = {}

    def add_student(self, name):
        self._grades[name] = []

    def report_grade(self, name, score):
        self._grades[name].append(score)

    def average_grade(self, name):
        grades = self._grades[name]
        return sum(grades) / len(grades)


book = SimpleGradebook()
book.add_student('Kho Sung Pil')
book.report_grade('Kho Sung Pil', 90)

print(book.average_grade('Kho Sung Pil'))

>>>
90.0
~~~
하지만 딕셔러니는 과도하게 사용할 경우 코드를 취약하게 할 가능성이 있다. 위의 예시에서 **SimpleGradebook()** 클래스를 확장해서 과목별로 성적을 저장할 때 _grades 딕셔너리를 변경해서 학생 이름(키)을 다른 딕셔너리(값)에 매핑한다.

~~~python
class BySubjectGradebook(object):
    def __init__(self):
        self.grades = {}
    
    def add_student(self, name):
        self._grades[name] = {}

    def report_grade(self, name, subject, grade):
        by_subject = self._grades[name]
        grade_list = by_subject.setdefault(subject, [])
        grade_list.append(grade)

    def average_grade(self, name):
        by_subject = self._grades[name]
        total, count = 0, 0
        for grades in by_subject.values():
            total += sum(grades)
            count += len(grades)
        return total / count


book = BySubjectGradebook()
book.add_student('pirunita')
book.report_grade('pirunita', 'Math', 75)
book.report_grade('pirunita', 'Math', 90)
book.report_grade('pirunita', 'Eng', 90)
book.report_grade('pirunita', 'Eng', 95)
~~~
그 다음 요구사항으로 수업의 최종 성적에서 각 점수가 차지하는 비중을 매긴다고 한다. 구현하는 방법 중 하나는 안쪽 딕셔너리를 변경하여 과목 - 성적 매핑이 아닌 *성적과 비중을 담은 튜플(score, weight)에 매핑한다.*

~~~python
class WeightedGradebook(object):
    def __init__(self):
        self.grades = {}
    
    def add_student(self, name):
        self._grades[name] = {}
    
    def report_grade(self, name, subject, score, weight):
        by_subject = self._grade[name]
        grade_list = by_subject.setdefault(subject, [])
        grade_list.append((score, weight))

    def average_grade(self, name):
        by_subject = self._grade[name]
        score_sum, score_count = 0, 0
        for subject, scores in by_subject.items():
            subject_avg, total_weight = 0, 0
            for score, weight in scores:
                # ...

book.report_grade('pirunita', 'Math', 80, 0.10)
~~~
이렇게 되면 클래스를 사용하는 방법이 더 어려워졌으며 위치 인수의 의미도 명확하지 않다. 따라서 이 때는 딕셔너리와 튜플 대신 **클래스의 계층 구조**를 사용한다. *딕셔너리를 담은 딕셔너리는 사용하면 안된다.* 데이터를 잘 캡슐화한 인터페이스를 제공하는 것이 좋다.

### 클래스 리팩토링 ###
&nbsp;&nbsp;의존 관계에서 가장 아래에 있는 성적부터 클래스로 옮긴다. 또한 성적은 변하지 않으니 튜플을 사용한다.
~~~python
grades = []
grades.append((95, 0.45))

total = sum(score * weight for score, weight in grades)
total_weight = sum(weight for _, weight in grades)
average_grade = total / total_weight
~~~
하지만 튜플은 위치에 의존하므로 더 많은 정보를 연관 지으려면 튜플을 사용하는 곳을 모두 찾아 아이템을 세 개 이상 쓰도록 수정한다.
~~~python
grades = []
graders.append((95, 0.45, 'Great job'))

total = sum(score * weight for score, weight, _ in grades)
total_weight = sum(weight for _, weight, _ in grades)
average_grade = total / total_weight
~~~
하지만 튜플을 점점 길게 확장하는 것은 딕셔너리 안의 딕셔너리와 비슷하므로 다른 방법을 고려한다.

**collection 모듈의 namedtuple 타입**을 고려하여 immutable data class(불변 데이터 클래스)를 쉽게 정의할 수 있다.
~~~python
import collections
Grade = collections.namedtuple('Grade', ('score', 'weight'))
~~~
불변 데이터 클래스는 위치 인수나 키워드 인수로 생성할 수 있으며 필드는 **이름이 붙은 속성**으로 접근할 수 있다. 이름이 붙은 속성은 추후에 동작을 추가할 때 namedtuple에서 직접 작성한 클래스로 쉽게 바꿀 수 있다. 하지만 namedtuple에도 단점이 존재한다.
* namedtuple로 만들 클래스에 기본 인수 값을 설정할 수 없다. 데이터에 선택적인 속성이 많으면 다루기 힘드므로 이때는 클래스를 직접 정의한다.
* namedtuple 인스턴스의 속성 값을 숫자로 된 인덱스와 순회 방법으로 접근할 수 있다. namedtuple 인스턴스를 사용하는 방식을 모두 제어할 수 없다면 클래스를 직접 정의하는 것이 좋다.

이제 성적들을 담은 단일 과목을 표현하는 클래스를 작성해 보자.
~~~python
class Subject(object):
    def __init__(self):
        self._grades = []

    def report_grade(self, score, weight):
        self._grades.append(Grade(score, weight))

    def average_grade(self):
        total, total_weight = 0, 0
        for grade in self._grades:
            total += grade.score * grade.weight
            total_weight += grade.weight
        return total / total_weight


class Student(object):
    def __init__(self):
        self._subjects = {}

    def subject(self, name):
        if name not in self._subjects:
            self._subjects[name] = Subject()
        return self._subjects[name]

    def average_grade(self):
        total, count = 0, 0
        for subject in self._subjects.values():
            total += subject.average_grade()
            count += 1
        return total / count


class Gradebook(object):
    def __init__(self):
        self._students = {}

    def student(self, name):
        if name not in self._students:
            self._students[name] = Student()
        return self._students[name]


book = Gradebook()
kho = book.student('kho sung pil')
math = kho.subject('Math')
math.report_grade(80, 0.10)
print(kho.average_grade())

>>> 81.5
~~~
코드가 다소 길어졌지만 훨씬 이해하기 쉬우며 클래스들을 사용하는 예제도 더 명확하고 확장하기 쉽다.

### 요약 ###
* 다른 딕셔너리나 긴 튜플을 값으로 담은 딕셔너리는 지양한다.
* 클래스의 데이터 유연성이 필요없다면 가벼운 컨테이너 namedtuple을 사용한다.
* 내부 상태를 관리하는 딕셔너리가 복잡해지면 여러 헬퍼 클래스를 사용한다.


<br>
<hr>
브렛 슬라킨, ⌜파이썬 코딩의 기술⌟, 김형철, 길벗(2017)