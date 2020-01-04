---
layout: post
title:  "[Hadoop] 1. Introducing the Hadoop"
date:   2020-01-04 12:00:00 +0900
lang: ko
tags: BigData
---
<hr>

## Hadoop ##

&nbsp;&nbsp;*지난 학기동안 빅데이터프로그래밍을 들었을 때 Hadoop을 처음 접하였다. 이 분야는 진입장벽이 높다고 볼 수 있으며 데이터베이스와 분산처리를 아는 것이 좋고 Java나 Python, Scalar도 익히는 것이 좋다. 일단 데이터베이스와 분산처리가 공부해야 할 양이 엄청나게 방대하고, 학교에서 Hadoop을 알려주는 곳은 많지 없으며(내가 알기로는..) 주로 회사나 독학으로 지식을 습득할 것이다. 나도 학교에서 처음 접하였고, <u>Hadoop definitive guide</u> 책과 강의 슬라이드를 많이 참조하였다.*

### 1. Big Data ###
![]({{ site.url }}/assets/img/hadoop/1_1.png){: width="70%" height="70%" .center}

&nbsp;&nbsp;Big Data란 무엇일까? 빅데이터는 우리 주변 어느 곳에나 산재한다. Hadoop definitive guide에는 다음과 같은 사례를 제시한다.
* 뉴욕증권거래소에서는 하루에 4.5TB의 데이터가 발생
* Facebook은 2,400억 개의 사진을 보유하고 있으며, 매달 7 petabytes 증가
* 인터넷 아카이브는 약 18.5 petabytes의 데이터를 저장

이렇게 엄청난 데이터를 저장하고 분석하는 것이 매우 어렵다. 왜 일까?
<br>
<u>그 동안 하드 디스크의 용량은 엄청나게 증가했지만 데이터를 읽는 속도는 따라가지 못했다.</u> 단일 디스크의 데이터를 읽는 데 너무 많은 시간이 걸리기 때문에 여러 개의 디스크에서 동시에 병렬적으로 데이터를 읽는 것이다. 이는 두 가지 사안을 고려해야 한다.

1. Hardware fault tolerance
   - Hardware를 많이 사용할 수록 장애가 발생할 확률 증가
   - 데이터 손실을 막기 위해 데이터를 여러 곳에 복제하는 방법 존재
   - RAID, HDFS ...

2. Merge with splited data
   - 분산 시스템에서 다중 출청의 데이터를 병합하면서 정합성을 지켜야 한다.
   - MapReduce는 디스크에서 데이터를 읽고 쓰는 문제를 key-value pair의 계산으로 변환된 추상화 프로그래밍 모델 제공

이러한 문제들을 어느 정도 완화시켜 주는 <u>오픈 소스 빅데이터용 프레임워크</u>인 **Hadoop**은 안정적이고 확장성이 높은 저장 및 분석 플랫폼을 제공한다고 볼 수 있다.

### 2. Why we need Hadoop for Big Data? ###
&nbsp;&nbsp;**Apache Hadoop**은 위에서 언급한 MapReduce 프로그래밍 모델을 사용하여 Bigdata의 dataset을 분산저장하고 처리하는 데 사용되는 open-source 프레임워크이다.

따라서 병렬컴퓨팅을 위한 computer cluster로 이루어져 있으며 Hadoop의 모든 모듈들은 <u>hardware 장애 발생과 이를 자동으로 핸들링</u>하기 위해 설계되었다.

* Query All Your Data
  - MapReduce는 전체가 한 번의 query로 대규모의 Dataset을 처리한다. MapReduce는 batch query processor(일괄 질의 처리기)이며, 전체 Dataset을 대상으로 ad hoc query(비정형 쿼리)수행하여 합리적인 시간에 결과를 보여준다.

### 3. MapReduce ###
&nbsp;&nbsp;**MapReduce**는 programming model이며 클러스터 환경에서 병렬 및 분산 알고리즘으로 large dataset을 처리하기 위해 설계 되었다. **Map**과 **Reduce**로 이루어져 있다.
* Map: performs typically filtering and sorting
* Reduce: performs typically a summary operation

각 단계는 Input과 Output으로 Key-value pair를 가지며 Map 함수와 Reduce 함수를 작성해야 한다. 다음은 MapReduce를 프로그래밍 하는 예시이다.

#### Python ####
~~~python
# Mapper
def getName(line):
    return (line.split('\t')[1], 1)

# Reducer
def addCounts(hist, (name, c)):
    hist[name] = hist.get(name, default=0) + c
    return hist

# Key-value iterators
input = open('employees.txt', 'r')
intermediate = map(getName, input)
result = reduce(addCounts, intermediate, {})
~~~


#### Java ####
{% highlight java %}
public class HistogramJob extends Configured implements Tool{
    // class FieldMapper
    public static class FieldMapper extends MapReduceBase implements Mapper<LongWritable,Text,Text,LongWritable>{
        private static LongWritable ONE = new LongWritable(1);
        private static Text firstname = new Text();

        @Override
        public void map(LongWritable Key, Text value,OutputCollector<Text,LongWritable> out, Reporter r){
            firstname.set(value.toString().split("\t")[1]);
            output.collect(firstname, ONE);
        }
    } 

    // class LongSumReducer
    public static class LongSumReducer extends MapReduceBase implements Mapper<LongWritable,Text,Text,LongWritable>{
        private static LongWritable sum = new LongWritable();

        @Override
        public void reduce(Text key, Iterator<LongWritable> vals, OutputCollector<Text,LongWritable> out, Reporter r){
            long s = 0;
            while(vals.hasNext()){
                s += vals.next().get();
            }
            sum.set(s);
            output.collect(key, sum);
        }
    }
} 
{% endhighlight %}


### 4. Data flow in MapReduce ###
* Job: 클라이언트가 수행하는 작업의 기본 단위
    - Input Data, MapReduce program, 설정 정보
* Task: Job에서 나누어 실행하는 작업
    - Map Task, Reduce Task
* Split: Hadoop이 MapReduce Job의 Input을 split이라 부르는 고정 크기 조각으로 분리
* Record: Split에서 아용자 정의 맵 함수로 처리하는 Data 단위

&nbsp;&nbsp;Hadoop은 partition된 Data가 저장된 클러스터의 각 머신에서 MapReduce 프로그램을 실행하며 이는 Hadoop의 **YARN**이라 불리는 Hadoop resource management system을 이용한다.

![]({{ site.url }}/assets/img/hadoop/1_2.png){: width="85%" height="85%" .center}

![]({{ site.url }}/assets/img/hadoop/1_3.png){: width="85%" height="85%" .center}

MapReduce는 기본적으로 batch 단위의 processing system이며 다양한 처리 패턴도 존재한다.
* Interactive SQL
* Iterative processing: Machine learning과 같은 반복 연산
* Stream processing: 실시간 프로세싱


