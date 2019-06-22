---
layout: post
title:  "[Docker] Container Image 실행하기"
date:   2019-04-22 00:00:00 +0900
lang: ko
tags: DevOps
---
## 컨테이너 이미지 실행하기 ##
<hr>

&nbsp;&nbsp;**docker run** 커맨드는 인터넷이 연결되어 있다면 자동으로 image를 찾아서 다운받고 구동한다.
예를 들어 fedora를 container에 띄우고 싶을 때
~~~
docker run fedora cat /etc/os-release
~~~
* cat /etc/os-release: container에 실행되고 있는 OS의 버전과 종류 확인
* 이미지 이름 뒤에 태그를 지정하지 않았으므로 :latest를 지정

### Image vs Container ###
* Image: Container instance가 영구적으로 저장된 것
  - docker images: 현재 시스템에 저장된 이미지를 볼 수 있다.
  - docker rmi <image_name>: 원하는 image 삭제
  - docker run: image 실행
* Container: Image를 실행하면 container가 생성되며 백그라운드에서 실행하면(-d 옵션) 커맨드 종료 후에도 계속 구동할 수 있다.
  - docker ps: 현재 실행 중인 컨테이너를 본다.
    + -a: container가 종료될 때 그 시점의 container 상태가 저장되며 그 때 -a를 붙여도 표시되지 않는다.
  - docker start <container_name>: 컨테이너 재구동
    + -i: container의 실행 결과가 로컬 셸로 전달된다.
    ~~~
    docker start -i 28172e7ffbc6
    ~~~
  - docker stop <container_name>: 컨테이너 멈춤

<hr>

### 인터렉티브 방식으로 container image 구동하기 ###
&nbsp;&nbsp;Container image를 구동하면, 지정한 커맨드를 실행한 뒤에 종료한다.
* -d: 웹 서버, 프린트 서비스와 같이 지속적으로 서비스를 제공하도록 커맨드를 백그라운드에서 계속 실행하게 한다.
* -i: 포그라운드에서 인터렉티브 방식으로 실행한다.
  - -t: 인터렉티브 방식으로 실행할 때는 -t 옵션을 지정해서 터미널 세션을 별도로 열어주면 좋다.

<br>

#### 배시 셸 구동하기 ####
&nbsp;&nbsp;Container 안에서 별도로 셸을 실행해 작업을 수행하려면 container를 인터렉티브 방식으로 구동하는 것이 좋으며 container 내부에서 내용을 수정할 수 있다.
~~~
docker run -it ubuntu /bin/bash
~~~
Container 내부에 bash shell을 띄웠으므로 어떤 프로세스가 실행되고 있는 지 확인 해 보자.
~~~
root@baf3c3803a19:/# ps -e
  PID TTY          TIME CMD
    1 pts/0    00:00:00 bash
   13 pts/0    00:00:00 ps
~~~
* ID가 1인 프로세스는 bash command
* 현재 실행되고 있는 프로세스는 ps command
* Container마다 별도의 프로세스 테이블을 갖기 때문에 host의 프로세스는 확인 불가능

Container 내부에 필요한 패키지를 설치한 후 exit를 통해 Container를 빠져나온다. 이 시점에서 Container를 image로 저장해두면 이 환경을 그대로 사용할 수 있다.
~~~
 khosungpil@khosungpil  ~/docker   master  docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                            PORTS               NAMES
baf3c3803a19        ubuntu              "/bin/bash"         5 minutes ago       Exited (127) About a minute ago

 khosungpil@khosungpil  ~/docker   master  docker commit -a "John" baf3c3803a19 testrun
~~~
이러면 로컬 시스템에 testrun이란 이름의 image가 생성되며 docker run을 통해 바로 사용할 수 있다.

<hr>

### 서비스를 Container로 만들어 실행하기 ###

&nbsp;&nbsp;호스트에서 서비스를 직접 구동하는 것보다 서비스를 **Container**에 담아 실행하는 편이 더 유리하다.
* 설정: 서비스를 Container 안에 담으면 서비스에 필요한 실행 파일, 라이브러리, 설정 파일 등 다양한 요소를 모두 설정할 수 있으며 다른 호스트에게 Image로 옮길 수 있다.
* 격리: Container마다 파일 시스템과 네트워크 인터페이스를 별도로 갖고 있기 때문에 동일한 서비스 Container를 원하는 수만큼 실행할 수 있다.

#### Container로 웹 서버 실행 ####
~~~
docker run -d -p 80:80 -p 443:443 --name=MyWebServer \
-v /var/www/:/var/www testrun \
/usr/sbin/httpd -DFOREGROUND
~~~
* -d: 데몬 모드로 실행, Container를 백그라운드에서 실행한다.
* -p 80:80, -p 443:443: 포트 매핑, Container의 포트를 호스트의 포트에 연결한다. 클론의 왼쪽은 호스트의 포트고 오른쪽은 Container의 포트, Container의 TCP 80 포트(HTTP)와 443번 포트(HTTPS)를 호스트의 동일한 포트로 연결
* --name=MyWebServer: Container 이름, Container ID 대신 지정한 이름을 사용할 수 있다.
* -v /var/www/ : /var/www/: 마운트 지점 지정, 호스트의 디렉토리(클론의 왼쪽)를 Container의 디렉토리(클론의 오른쪽)에 마운트한다. -v 옵션을 사용해 apache의 default directory에 있는 공유 웹 콘텐츠를 사용하도록 지정
* testrun: Image, 예제에서 사용한 Image 이름
* /usr/sbin/httpd -DFOREGROUND: 커맨드, httpd 데몬에 -DFOREGROUND 옵션을 지정해 실행

따라서 호스트의 모든 IPV4 네트워크 인터페이스의 TCP 80번과 443번 포트로 들어오는 요청은 Container의 동일한 포트로 전달된다.

구동중인 Container 내부를 탐색핫기 위해서 bash를 다시 실행할 수 있다.
~~~
docker exec -it MyWebServer /bin/bash
~~~

#### Container에서 실행하는 서비스에서 사용할 resource 제한 ####
&nbsp;&nbsp;Container의 default는 메모리와 CPU의 사용량을 제한하지 않는다. --memory, --memory-swap, --cpu-shares, --cpuset-cpus 같은 옵션으로 메모리 용량과 CPU 개수에 제한을 걸 수 있다.

~~~
docker run -d -p 80:80 -p 443:443 --name=LimitedWebServer \
-v /var/www/ : /var/www/ --memory=10m -memory-swap=-1 \
--cpu-shares=256 testrun /usr/sbin/httpd -DFOREGROUND
~~~
* 여기서 Container는 10MB의 RAM만 사용할 수 있다. memory-swap이 지정되어 있지 않다면 container는 메모리의 두 배 공간을 swap 공간으로 사용할 수 있으며 30m이라고 정하면 최대 30MB만큼 swap 공간으로 사용할 수 있다.

* --cpu-shares가 지정되어 있지 않으면 모든 container가 동등하게 cpu를 사용한다. default는 전체(1024)를 사용할 수 있다.

* --cpuset-cpu: 시스템에 있는 CPU 중에서 사용할 CPU 셋을 지정할 수 있다.
  - --cpuset-cpus=0,1: CPU셋 0이나 1을 사용
  - --cpuset-cpus=0,1: 세 번째 CPU 셋을 사용
  - --cpuset-cpus=0,1: CPU 셋 1이나 2나 3을 사용