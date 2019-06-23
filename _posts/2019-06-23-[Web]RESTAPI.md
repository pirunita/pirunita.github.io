---
layout: post
title:  "[Web] REST"
date:   2019-06-23 12:00:00 +0900
lang: ko
tags: Backend
---
## REST ##

* REST: REpresentational State Transfer, 자원을 정의하고 자원의 정보(상태)를 주고받는 모든 것
    - Browser와 같은 Web Client는 Resource identifier를 통해 resource에 대해 어떤 요청을 하고 그 결과를 받아 브라우저 화면에 띄우는 방식으로 표현되며 이 과정은 **브라우저 표현상태(Representational state)**의 변경(Transfer)을 일으킨다. **REST**는 이러한 특성을 표현한다.

### Why REST ? ###
* Application 분리 및 통합
* 서버프로그램은 다양한 Web Browser와 OS에 통신 가능해야 한다, 즉 인터페이스 범용성과 규모 확장성
* 중간적 구성요소를 통한 응답 지연 감소, 보안 강화

### REST 구성 요소 ###
1. 자원(Resource): URI
    * 모든 자원에 고유 ID가 존재하며 자원은 Server에 존재
    * 자원을 구별하는 ID는  ‘/groups/:group_id’와 같은 HTTP URI
    * Client는 URI를 이용해서 자원을 지정하고 해당 자원의 상태(정보)에 대한 조작을 Server에 요청
2. 행위(Verb): HTTP Method
    * HTTP 프로토콜의 Method를 사용
    * HTTP 프로토콜은 GET, POST, PUT, DELETE 같은 메서드 제공
3. 표현(Representation of Resource)
    * Client가 자원의 상태에 대해서 조작을 요청하면 Server는 응답
    * REST에서 하나의 자원은 JSON, XML, TEXT, RSS 등 여러 형태의 Representation으로 나타내어 지며 일반적으로 JSON, XML을 통해 데이터를 주고 받는다.

### REST 특징 ###
1. Server-Client(서버-클라이언트 구조)
    * 자원이 있는 쪽이 **Server**, 자원을 요청하는 쪽이 **Client**
        - REST Server: API를 제공, 비즈니스 로직 처리 및 저장
        - Client: 사용자 인증, context 관리
    * Dependency 감소
2. Stageless(무상태)
    * 작업을 위한 상태정보를 따로 저장하고 관리하지 않음
    * API Server는 각각의 요청을 별개로 인식하고 처리
        - 각 API Server는 Client의 요청을 단순 처리
        - 이전 요청이 다음 요청 처리에 연관되선 안됨
    * Client의 context를 Server에 저장하지 않음
        - 세션, 쿠키와 같은 context 정보를 신경쓰지 않아도 되므로 구현 단순화
3. Cacheable(캐시 처리 가능)
    * 웹 표준 HTTP 프로토콜을 그대로 사용하므로 웹에서 사용하는 기존의 인프라 활용
        - HTTP의 캐싱 기능 적용
        - Last-Modified 태그, E-Tag를 이용하면 캐싱 구현 가능
    * 대량의 요청을 효율적으로 처리하기 위해 캐시 요구
    * 캐시 사용을 통해 응답시간 단축, REST Server 트랜잭션이 발생하지 않아 성능, 서버 자원 이용률 향상
4. Layered System(계층화)
    * REST Server는 다중 계층으로 구성 가능
        - REST API Server는 순수 비즈니스 로직을 수행, 그 앞단에 보안, 로드밸런싱, 암호화, 사용자 인등을 추가하여 구조의 유연성 확장
        - PROXY, Gateway 같은 네트워크 기반의 중간매체 사용 가능
5. Code-On-Demand
    * Server로부터 스크립트를 받아서 Client에서 실행
6. Uniform Interface(인터페이스 일관성)
    * URI로 지정한 자원에 대한 조작을 통일되고 한정적인 인터페이스로 수행
    * HTTP 표준 프로토콜을 따르는 모든 플랫폼에서 사용 가능
<hr>

## REST API ##
* API(Application Programming Interface): Data와 기능의 집합을 제공하여 컴퓨터 프로그램 간 상호작용을 촉진하고 서로 정보를 교환 가능하도록 함

* REST API: REST 기반으로 서비스 API 구현

### REST API 특징 ###
* 시스템을 REST 기반으로 시스템을 분산하여 *확장성*과 *재사용성*을 높여 유지보수 및 운용 편리
* REST는 HTTP 표준을 기반으로 구현하여 HTTP를 지원하는 언어로 클라이언트, 서버를 구현

### REST API 설계 기본 규칙 ###
* document: 객체 인스턴스나 데이터베이스의 레코드같은 개념
* collection: 서버에서 관리하는 directory resource
* store: client에서 관리하는 resource 저장소
1. URI는 정보의 자원을 표현
    * resource는 명사, 소문자를 사용
    * resource의 document 이름은 단수 명사 사용
    * resource의 collection 이름은 복수 명사 사용
    * resource의 store 이름은 복수명사 사용
2. 자원에 대한 행위는 HTTP Method(GET, PUT, POST, DELETE)로 표현
    * URI에 HTTP Method가 들어가면 안됨
        - Ex) GET /members/delete/1 -> DELETE /members/1
    * URI에 행위에 대한 표현이 들어가면 안됨
        - Ex) GET /members/show/1 -> GET /members/1
        - Ex) GET /members/insert/2 -> POST /members/2
    * 경로 부분 중 변하는 부분은 유일한 값으로 대체(:id는 하나의 특정 resource를 나타내는 고유값)
        - Ex) student를 생성하는 route: POST/students
        - Ex) id=12인 student를 삭제하는 route: DELETE/studnets/12

| METHOD | 역할 |
| :--------- | :--------- |
| POST       | POST를 통해 해당 URI를 요청하면 리소스를 생성 |
| GET        | GET를 통해 리소르를 조회한다. 리소스를 조회하고 해당 도큐먼트에 대한 자세한 정보를 가져온다. |
| PUT        | PUT를 통해 해당 리소스를 수정 |
DELETE       | DELETE를 통해 리소스 삭제 |

### REST API 설계 규칙 ###
1. 슬래시 구분자(/)는 계층 관계를 나타낸다.
2. URI 마지막 문자로 슬래시(/)를 포함하지 않는다.
    * REST API는 URI를 만들어 통신을 하므로 혼동을 주지 않기 위해 URI 경로 마지막에 슬래시(/)를 사용하지 않는다.
3. 하이픈(-)은 URI 가독성을 높임
4. 밑줄(_)은 URI에 사용하지 않음
5. URI 경로는 소문자가 적합
6. 파일확장자는 URI에 포함하지 않음
    * Accpet header를 사용
    * Ex) https://restapi.example.com/members/soccer/345/photo.jpg (x)
    * Ex) GET / members/soccer/345/photo HTTP/1.1 Host: restapi.example.com Accept: image/jpg (o)

7. 리소스 간에 연관 관계가 있는 경우
    * /리소스명/리소스, ID/관계가 있는 다른 리소스 명
    * Ex) GET : /users/{userid}/devicess

<hr>

## RESTful ##
* RESTful: 일반적으로 REST라는 아키텍처를 구현하는 웹 서비스를 나타내는 용어

### Why RESTful? ###
* 이해하기 쉽고 사용하기 위운 REST API를 만드는 것



## Reference ##
* https://meetup.toast.com/posts/92
* https://12bme.tistory.com/25
* https://gmlwjd9405.github.io/2018/09/21/rest-and-restful.html