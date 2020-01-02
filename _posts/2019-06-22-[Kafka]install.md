---
layout: post
title:  "[Kafka] Start"
date:   2019-06-22 15:00:00 +0900
lang: ko
tags: Backend
---
<hr>

## Apache Kafka ##

&nbsp;&nbsp;Apache Kafka는 대용량, 대규모의 메시지 데이터를 빠르게 처리할 수 있는 메시지 큐이며 분산처리에 특화되어 있다.

Kafka는 PUB/SUB(Publishier, Subscriber) 모델로 메시지를 전달하고 받는 구조다. Publisher는 메시지를 topic을 통해 카테고리화하고 Subscriber는 해당 topic을 subscribe하여 메시지를 읽는다. 

![]({{ site.url }}/assets/img/kafka/kafka1.png){: width="70%" height="70%" .center}


Kafka는 다음과 같이 구성되어 있다.

### Topic & Partition ###
* Topic: 메시지는 topic으로 분류되고 topic은 여러 파티션으로 나눠진다. 메시지의 상대적 위치를 나타내는 것이 **offset**이다. 
* Partition: 하나의 topic에 여러 partition을 통해 메시지를 분산 저장하여 병렬처리를 한다.


### Producer & Consumer ###
* Producer: 메시지를 생산하는 주체, Topic에 메시지 작성
* Consumer: 메시지를 소비하는 주체, Subscribe 후에 메시지를 소비할 수 있다. 소비를 하는 표시는 topic 내의 각 partition에 존재하는 offset의 위치를 통해 이전에 소비한 offset 위치를 기억하고 관리한다.

### Consumer Group ###
반드시 해당 topic의 partition은 consumer group과 1:n 매칭을 해야 한다. 따라서 partition을 늘릴 때 consumer group 내의 consumer 개수도 고려한다. 보통은 맞추는 것을 권장한다.

### Broker, Zookeeper ###
* Broker: Kafka Server, 동일한 노드 내에서 여러 broker server를 띄울 수 있다.
* Zookeeper: 분산 메시지 큐의 정보를 관리 해 주는 역할, 반드시 실행해야 한다.

### Replication ###
Broker server를 복제한다.


<hr>

## 설치 환경 ##

~~~
OS: Mac OS Mojave 10.14.5
~~~

<hr>

## Install ##
1. 다음 링크에 접속한 후 Binary를 다운로드한다. 나는 Kafka 2.2.1 패키지의 Scala 2.12를 다운받았다.
https://kafka.apache.org/downloads
![]({{ site.url }}/assets/img/kafka/kafka2.png)

2. Kafka는 Scala로 개발되었으므로 JVM이 설치되어야 한다. 난 이미 깔려 있으니 생략한다.

3. Kafka를 다운받고 압축을 해제한 후 폴더 내에는 다음과 같이 있다.

~~~
khosungpil@khosungpil > ~/kafka_2.12-2.2.1 > > master > ls
LICENSE   NOTICE    bin       config    libs      site-docs
~~~

<hr>

## Usage ##

### ZooKeeper setting & run
1. 다음을 입력하면 Zookeeper가 실행된다.
~~~
bin/zookeeper-server-start.sh config/zookeeper.properties
~~~
2. 정상적으로 실행되면 다음 메시지가 뜬다.
~~~
[2019-06-22 14:46:24,752] INFO binding to port 0.0.0.0/0.0.0.0:2181 (org.apache.zookeeper.server.NIOServerCnxnFactory)
~~~

### Kafka run ###
1. ZooKeeper를 실행한 상태에서 kafka를 동작한다.
~~~
bin/kafka-server-start.sh config/server.properties
~~~
2. 정상적으로 실행되연 다음 메시지가 뜬다.
~~~
[2019-06-22 14:55:46,259] INFO Kafka version: 2.2.1 (org.apache.kafka.common.utils.AppInfoParser)
[2019-06-22 14:55:46,261] INFO Kafka commitId: 55783d3133a5a49a (org.apache.kafka.common.utils.AppInfoParser)
[2019-06-22 14:55:46,279] INFO [KafkaServer id=0] started (kafka.server.KafkaServer)
~~~

### Topic manage ###
1. Topic을 생성한다.
~~~
bin/kafka-topics.sh -create -zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
~~~
~~~
Created topic test.
~~~
2. 2181 port로 열었기 때문에 다음과 같이 Topic 리스트를 확인한다.
~~~
bin/kafka-topics.sh --list --zookeeper localhost:2181
~~~
~~~
test
~~~
3. Topic을 삭제한다.
~~~
bin/kafka-topics.sh --delete --zookeeper localhost:2181 --topic test
~~~
~~~
Topic test is marked for deletion.
Note: This will have no impact if delete.topic.enable is not set to true.
~~~

### Message send & receive ###
1. Producer를 실행한다. test라는 topic을 subscribe하는 consumer에게 보낸다고 broker에게 메시지를 보낸다.
~~~
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test
~~~
2. Consumer를 실행한다. Producer와 같은 port를 쓴다.
~~~
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
~~~
3. Producer 콘솔창에 메시지를 전송하면 Consumer 콘솔창에 메시지가 수신되어 나타난다.

4. 다음은 전체 예시이다.
![]({{ site.url }}/assets/img/kafka/kafka3.png)
<hr>

## Reference ##
https://yfkwon.tistory.com/26

https://medium.com/@umanking/%EC%B9%B4%ED%94%84%EC%B9%B4%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-%EC%9D%B4%EC%95%BC%EA%B8%B0-%ED%95%98%EA%B8%B0%EC%A0%84%EC%97%90-%EB%A8%BC%EC%A0%80-data%EC%97%90-%EB%8C%80%ED%95%B4%EC%84%9C-%EC%9D%B4%EC%95%BC%EA%B8%B0%ED%95%B4%EB%B3%B4%EC%9E%90-d2e3ca2f3c2