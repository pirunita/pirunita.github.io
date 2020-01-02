---
layout: post
title:  "[Docker] Docker 기초 및 command 정리"
date:   2019-04-11 00:00:00 +0900
lang: ko
tags: DevOps
---
<hr>

## Docker basic & Command ##


### 왜 Docker를 사용하는가? ###
&nbsp;&nbsp;Containers는 하나의 서로 다른 프로그램이나 프로세스를 고립시키는 방법이며, 프로그램이 오류없이 빠르게 deploy를 시킬 수 있게한다. 또한 application을 호스트 컴퓨터에 직접 설치할 때보다 높은 유연성을 제공하며 가상머신과 비교했을 때 CPU, 메모리, 디스크 공간과 같은 시스템 리소스를 적게 소비한다.

<br>

### Docker 이미지와 컨테이너 ###
* Containerization: Application을 하나의 컨테이너 단위로 구동하는 데 필요한 모든 구성 요소(라이브러리, 설정 파일, 실행 파일 등..)를 하나로 묶는다. 이러한 단위를 **image**라고 한다.

* Image: 도커가 설치되고 구동할 준비를 하고 있는 로컬 파일 시스템, 또는 repository에 저장된 하나의 정적인 단위.
  - Image를 파일시스템에 저장하면 tarb4all(tar.gz) 파일 형태로 저장된다.

* Container: Docker Image를 구동한 instance를 의미한다.

<br>

### Docker command ###
* Docker 구성 요소에 대한 정보 조회
  - docker version: Docker의 버전 정보
  - docker info: Docker를 구동하는 시스템에 대한 정보
  - docker comand: Docker command에서 사용할 수 있는 subcommand와 option을 보여준다.
  - docker history: **Image**에 대한 히스토리 정보를 보여준다.

* 실행 중인 Container 다루기
  - docker ps: 현재 실행 중인 container list를 보여준다.
    + -a: 모든 container의 상태를 보여준다.
  - docker attach: 실행 중인 cantainer에 다른 command를 붙인다.
  - docker exec: 현재 실행 중인 컨테이너에서 command를 실행한다.
  - docker inspect: container의 metadata를 본다.
  - docker cp: container에 있는 파일을 호스트 시스템으로 복사
  - docker diff: container를 구동한 후에 컨테이너의 파일시스템에서 변경된 사항을 확인

* Image 다루기
  - docker images: 시스템에 있는 image 리스트를 보여준다.
  - docker run: image를 실행한다.
    + -rm: 프로세스가 종료되면 컨테이너 자동 삭제
    + -it: container 내부에 들어가기 위해 bash 키보드 입력 설정 추가
    + /bin/bash: bash쉘 실행
    ~~~
    docker run --rm -it <image_name> /bin/bash
    ~~~
  - docker pull: 레지스트리에서 image를 가져온다.
    + docker pull ubuntu:latest
  - docker push: 레지스트리에 image를 올린다.
  - docker load: 로컬 시스템에 tarball 형태로 존재하는 image를 불러온다.
  - docker export: container의 파일시스템을 로컬 시스템에 tarball 형태로 내보낸다.

* Docker 레지스트리 다루기
  - docker search: 레지스트리에서 이미지를 검색한다.(자신의 계정에 image를 올리거나 가져올 때)
  - docker login: 도커 허브 레지스트리 로그인
  - docker logout: 도커 허브 레지스트리 로그아웃

* 기존 이미지 수정하기
  - docker tag: image에 이름을 붙인다.
  - docker rename: image의 이름을 변경한다.

* container 상태 변경하기
  - docker pause: 실행 중인 container 일시 정지
  - docker unpause: 일시 정지한 이미지를 다시 시작
  - docker kill: container에게 kill 신호를 보냄 
  - docker restart: container를 멈췄다가 다시 시작

* docker 상태 관찰하기
  - docker events: 도커 서버에서 발생한 이벤트 확인
  - docker top: container의 프로세스 현황 확인
  - docker logs: container에서 생성한 로그 메시지 확인
  - docker stats: container에 대한 CPU 및 메모리 사용량 확인
  - docker wait: container가 멈출 때까지 관찰하며 container의 종료 코드를 화면에 표시

* Image나 container 생성하기
  - docker build: Image를 새로 생성
    + -t: image name을 붙임
    + dockerfile이 같은 폴더 내에 위치할 경우
    ~~~
    docker build -t <image_name> ./
    ~~~
  - docker rmi: Image를 제거
    ~~~
    docker rmi <image_name>
    ~~~
    + none으로 지정된 이름을 제거하기 위해
    ~~~
    docker rmi $(docker images -a|grep "<none>"|awk '{print $3}')
    ~~~

* 자주 사용하는 옵션
  - d: detached mode, 백그라운드 모드
  - p: 호스트와 container의 포트를 연결(포워딩)
  - v: 호스트와 container의 디렉토리를 연결(마운트)
  - e: container 내에서 사용 할 환경변수 설정
  - name: container 이름 설정
  - rm: 프로세스 종료 시 container 자동 삭제
  - it: 터미널 입력을 위한 옵션
  - link: container 연결
