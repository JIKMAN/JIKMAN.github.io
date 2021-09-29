---
title:  "[Hadoop] 맵 리듀스의 개념/동작"

categories:
  - IT
tags:
  - [HDFS, hadoop, map reduce]
toc: true
toc_sticky: true
---

![img](https://blog.kakaocdn.net/dn/deXi2i/btqDDXL5HQa/fZombbSSvQq9rJCKSs7b1k/img.png)



### **✔️**MapReduce 핵심 개념

맵 리듀스는 구글 내부에서 크롤링 된 문서, 로그 등 방대한 양의 raw data를 분석하는 과정에서 느낀 불편함에서 출발했습니다. 프로그램 로직 자체는 단순한데 입력 데이터의 크기가 워낙 방해서서 연산을 하나의 물리 머신에서 수행할 수가 없었습니다. 이 거대한 인풋 데이터를 쪼개어 수많은 머신들에게 분산시켜서 로직을 수행한 다음 결과를 하나로 합치자는 것이 핵심 아이디어 입니다.

 

 MapReduce 프레임워크에서 개발자가 코드를 작성하는 부분은 map과 reduce 두 가지 함수입니다. map은 전체 데이터를 쪼갠 청크에 대해서 실제로 수행할 로직입니다. 텍스트 데이터라면 텍스트를 파싱해서 단어의 개수를 세는 것이 예입니다. reduce는 분산되어 처리된 결과 값들을 다시 하나로 합쳐주는 과정이며, 이 역시 분산된 머신들에서 병렬적으로 수행됩니다.

 

정리하자면 다음과 같은 순서로 **MapReduce**가 진행됩니다.

**1. 쪼개기(Split):** 크기가 큰 인풋 파일을 작은 단위의 청크들로 나누어 분산 파일 시스템(ex. HDFS)에 저장합니다.

**2. 데이터 처리하기(Map)**: 잘게 쪼개어진 파일을 인풋으로 받아서 데이터를 분석하는 로직을 수행합니다.

**3. 처리된 데이터 합치기(Reduce)**: 처리된 데이터를 다시 합칩니다.

 

아래는 실제로 이런 분산 처리가 어떻게 진행되는 지를 나타내주는 다이어그램입니다. 



![img](https://blog.kakaocdn.net/dn/dV6SXC/btqDp19DbDq/JqxLqaJjzYwMHuHGHAIdgK/img.png)



**(1) fork**

인풋 파일은 미리 쪼개져서 분산 파일 시스템에 저장되어 있다고 가정하겠습니다. 사용자가 map과 reduce 함수를 정의한 프로그램을 실행시켜서 프로세스를 실행하면 이는 마스터 노드, map을 실행할 워커 노드(이하 Mapper 노드), reduce를 실행할 워커 노드(이하 Reducer 노드)들에 복사됩니다.

 

**(2) assign map and reduce**

마스터 노드는 Mapper워커들에게 mapping 역할을, reducer 노드들에게는 reduce 역할을 수행하라고 지정해줍니다. 이 때, mapper 노드의 개수나 reducer 노드의 개수는 사용자가 설정할 수 있습니다.

 

**(3)(4) read, map, local write**

Mapper 노드들은 쪼개어진 데이터 청크를 분산 파일 시스템으로부터 읽어옵니다. 그 다음 이 데이터에 Map 함수를 실행하여 K:V 형태의 Intermediate 데이터를 생성하고 이를 자기 자신의 로컬 디스크에 저장합니다. 그 다음 Mapper 노드는 마스터 노드에게 Mapping 작업을 모두 완료하였다고 알려줍니다.

 

**(5)(6) reduce, write** 

그러면 마스터 노드는 리듀서 노드들에게 reduce를 시작하라고 명령을 내려줍니다. reducer 노드들은 mapper 노드들의 디스크 공간에 저장되어 있는 Intermediate 데이터를 읽어옵니다. 그리고 reduce 함수를 실행하여 최종 결과물을 산출한 다음 파일 형태로 데이터를 출력합니다.

### **✔️**Word Counting Example

가장 유명한 예시인 워드 카운팅을 통해서 MapReduce에 대한 이해를 높여보도록 하겠습니다. 3대의 Mapper와 4대의 Reducer 노드로 이루어진 클러스터에서 워드 카운팅을 수행하는 상황입니다.



![img](https://blog.kakaocdn.net/dn/caP7BN/btqDsPmUFMr/DFGrLkokDFjNj0v7VmL0O0/img.png)



#### **Splitting, Mapping**

먼저input 텍스트 파일을 쪼개어 분산 파일 시스템에 저장합니다. 쪼개어진 텍스트 데이터는 3대의 Mapper노드에 각각 전달됩니다. 그리고 각 노드들은 Map 함수를 실행하여 입력으로 들어온 텍스트 데이터를 파싱하여 Dear:1, Bear:1, River:1 과 같은 Intermediate Key Value를 생성합니다.

#### **Shuffling**

Mapping으로 생성된 Intermediate Key Value 값을 Reducer 노드에 입력으로 전달하기 이전의 과정을 통틀어서 Shuffling이라고 부릅니다. (다소 복잡한 개념이므로 상세히 설명하도록 하겠습니다.) 먼저 Shuffling이 필요한 이유부터 알아보겠습니다.

 

우리가 MapReduce를 통해서 알고 싶은 것은 전체 텍스트에서 특정 단어가 얼마나 자주 등장했는지 입니다. 이를 위해서 전체 텍스트 데이터를 3조각으로 나누었고, 이를 Mapper 노드들에 분산 시켜 워드 카운팅을 진행했습니다. 문제는 이렇게 생성된 Intermediate Key Value 들에서 노드는 서로 다르지만 같은 키 값을 지닌 쌍들이 있다는 것입니다.

 



![img](https://blog.kakaocdn.net/dn/c3QFGp/btqDqjvCSvl/EPb5kbYWmySpbKJpSW6nB0/img.png)



예를들어 Dear라는 단어는 1번 Mapper와 3번 Mapper의 Intermediate Key Value에는 Dear:1이라는 값이 있습니다. 이를 적절히 Reduce 해주기 위해서는 이 값들이 모두 동일한 Reducer에게 전달이 되어야만합니다. ***즉, 동일한 키 값을 지닌 Intermediate Key Value Pairs는 동일한 Reducer에게 전달되어야 하며, 이 매핑 과정을 Shuffling이라고 합니다.***

 

그렇다면 셔플링은 구체적으로 어떠한 과정을 통해서 이루어질까요? 셔플링 과정을 좀 더 자세히 들여다 보겠습니다.



![img](https://blog.kakaocdn.net/dn/cH5qQT/btqDqipYWvL/DZnOZmOKV055loAZKaVXqK/img.png)



map 함수를 거쳐서 생성된 intermediate key value pairs는 먼저 메모리 상의 버퍼에 저장됩니다. 그 다음 partitioning이라는 과정을 거칩니다. 파티셔닝이란 해당 키가 처리되어야 하는 reduce task를 기준으로 데이터를 구분해줍니다. 보통 키 값을 해쉬하여 파티션을 결정합니다. 하둡에서는 기본적으로 키 값을 리듀서의 개수로 해쉬하는 방식을 사용합니다. (ex. key mod R(num of reducers)) 이렇게 해주면 각각의 키 값들이 어느 리듀서에서 처리되어야 하는 지를 구분할 수 있습니다. 각각의 파티션은 다시 키 값을 기준으로 소팅을 거쳐서 Mapper 노드의 local disk 공간에 저장됩니다.

 

*ps. 리듀서의 개수를 기준으로 키 값을 단순히 해쉬하게 되면 빈번하게 등장하는 단어 처럼 특정 reduce 작업의 부하가 높아질 우려가 있습니다. 그러므로 하둡은 직접 파티셔닝 로직을 작성할 수 있는 기능을 제공합니다.*

 

이렇게 파티셔닝 된 데이터는 이제 Combining이라는 과정을 거칩니다. Combining이란 파티션 내의 Key Value 데이터들에 대해서 리듀스 작업을 진행하여 리듀서 노드들의 부하를 줄여주는 작업을 말합니다. 워드 카운팅의 예시를 들어보면 아래와 같습니다. 



![img](https://blog.kakaocdn.net/dn/emzaWq/btqDsNW3LYU/LQ4N7KsLpq4ncLTfi02cIk/img.png)



컴파이닝 과정을 거친 데이터는이제 Mapper 노드의 local disk 공간에서 저장됩니다. 그리고 Mapping 과정을 완료했다는 정보와 파티셔닝 된 데이터의 위치 정보를 마스터 노드에게 알려주게 됩니다. 마스터 노드는 Mapper들로부터 전달받은 정보들을 취합하여 Reducer 노드들에게 명령을 내립니다. Reducer 노드들은 이제 각각의 Mapper 노드들에서 자신에게 할당된 파티션에 해당하는 데이터만 읽어와 reduce 작업을 진행함으로써 셔플링 과정이 마무리 됩니다.

 

셔플링 과정을 정리하면 다음과 같습니다.

**0. Mapping**: map의 결과물로 intermediate key value가 생성됨

**1. Partitioning**: 각각의 키 값을 기준으로 어느 리듀서에서 처리되어야 하는지 파티션을 결정함

**2. Sorting**: 같은 파티션 내의 값들은 키 값을 기준으로 소팅

**3. Combining**: 파티션 내의 값들에 대해 미리 reduce 작업을 진행하여 데이터의 양을 줄임

**4. Spill to locak disk**: 처리된 값들을 Mapper 노드의 디스크 공간에 저장

**5. Notify Master**: 매핑 과정을 완료하였다는 알림과 파티셔닝 정보를 마스터 노드에게 알림

**6. Notify Reducers**: 마스터 노드는 리듀서 노드들에게 이 정보를 전달

**7. Remote Read**: 리듀서 노드들은 자신에게 할당된 파티션들만 매퍼들에게서 읽어와서 리듀스 작업을 준비

### Reduce, Write

다시 원래의 워드 카운팅의 사례로 돌아오겠습니다. 셔플링 과정을 훌륭히 마친 데이터를 각각의 리듀서가 읽어와서 단어별 개수 카운팅을 완료하였습니다. 그리고 그 결과를 파일 형태로 출력하게 됩니다. 여기서 또 한 가지 문제가 있는데 리듀서들 역시도 분산된 노드들이라 리듀스 결과 데이터가 분산되어 있다는 것입니다. 이를 다시 어떻게 취합하여 저장을 하는 것일까요? 아래 다이어그램은 리듀스 결과를 마친 데이터가 어떻게 저장되는 지를 보여줍니다.



![img](https://blog.kakaocdn.net/dn/pRE5u/btqDvxTgnRG/cyTb6gOPX4QmT9XhwfOpeK/img.png)



MapReduce는 reduce 결과로 생성된 Key Value 값들을 별도로 취합하지 않고 input file을 쪼개어 저장했을 때 처럼 분산 파일 시스템에 저장합니다. 이는 reduce 결과물들을 다시 취합해야하는 수고를 덜어주고, 결과 데이터들을 가지고 다시 MapReduce 작업을 진행하기가 편리합니다. 또한 읽기 작업을 수행할 때에는 분산 파일 시스템이 이 역할을 해주게 되어 MapReduce 자체의 구현이 더 간결해지는 장점이 있습니다.

### **✔️**Reallife MapReduce

앞서 살펴본 워드 카운팅의 사례 외에도 MapReduce를 적용할 수 있는 사례들이 논문에 언급되어 있습니다. 앞서 언급했듯이 MapReduce 프로그래밍 모델에서 개발자는 Map과 Reduce 함수를 작성하게 되며, 그 이외의 복잡한 분산 처리 및 저장을 MapReduce 프레임워크가 수행해주게 됩니다. 아래 사례들에서 K는 Key, V는 Value를 의미합니다.

**1. Distributed Grep**

· Map: 주어진 텍스트 데이터 청크에서 정규표현식 패턴 매칭 수행, 패턴(K):1(V) 생성

· Reduce: intermediate value를 복사하여 그대로 저장(패턴 매칭만 수행하므로 별도의 sum 같은 로직을 수행하지 않는다.)

**2. Count URL Access Frequency**

· Map: 주어진 웹 페이지 요청 로그 데이터 청크에서 URL(K):1(V) 생성

· Reduce: 찾아낸 K/V들을 모두 합쳐서 URL(K): total count(V) 생성

**3. Reverse Web-Link Graph**

· Map: 웹 페이지 html 데이터 청크에서 그 안에 포함된 URL을 찾아냄. Target(찾아낸 URL): Source(원본 URL) 생성

· Reduce: 찾아낸 K/V를 합쳐서 Target: list(Source) 형태의 데이터 생성

**4. Term-Vector per Host**

term vector란 문서 혹은 일련의 문서들에서 가장 중요한 단어들을 word(K): frequency(V) 형태로 계산한 것입니다.

· map: 문서 데이터 청크에서 각 단어별 빈도수를 계산하여 hostname(K): term vector(V)를 계산합니다.(term vector 자체도 K:V 데이터 이므로 K:(K:V) 형태가 되겠네요)

· reduce: 앞서 구한 K/V 데이터를 모두 합쳐서 호스트 별로 term vector를 구합니다. hostname(K): term vector(V)

**5. Inverted Index**

· map: 웹 문서 데이터 청크를 입력 받아서 파싱한 다음 word(K): document ID(V) 형태의 데이터를 만듭니다.

· reduce: 앞서 구한 K/V 데이터를 입력 받아서 다음과 같은 형태로 합쳐줍니다. word: list(document ID) 특정 단어가 쿼리로 들어오면 이렇게 구한 list(documentID)를 리턴해주면 간단한 검색 엔진을 구현할 수 있습니다.

**6. Distributed Sort**

· map: 레코드에서 소팅의 기준이 되는 키 값을 골라내어 key(K): record(V) 형태의 데이터를 만듭니다.

· reduce: 리듀스에서는 앞서 구한 K/V를 그대로 합쳐주어 파일에 저장합니다. 왜냐하면 Map 함수를 거쳐 생성한 intermediate file 들은  이미 소팅이 되어 있기 때문입니다.

### **✔️**MapReduce Limitation

**1. 복잡한 셔플링**

MapReduce는 훌륭한 분산 데이터 처리 메커니즘을 보여주지만 한계점 역시도 존재합니다. 앞서 살펴보았듯이 Map과 Reduce 과정은 상당히 단순합니다. 그러나 이 둘 사이를 연결해주는 Shuffling 과정이 복잡하며 속도 또한 느린 것으로 악명이 높다고 합니다. 그래서 하둡 엔지니어들 사이에서는 MapReduce를 small Map, large Shuffle, small Reduce라고 불러야 한다는 농담이 있다고 합니다.

 

**2. 잦은 파일 입출력**

또한 각각의 노드들은 map이나 reduce 작업을 하기 위해서 파일을 읽고 쓰는 작업을 합니다. Mapper는 분산 파일 시스템에서 데이터를 읽어와서 intermediate key value를 생성한 뒤, 자신의 디스크 공간에 파일을 씁니다. Reducer는 다시 이 파일을 읽어와서 리듀스 작업을 진행한 다음 결과 파일을 분산 파일 시스템에 출력합니다. 이렇게 반복적인 disk I/O는 성능 저하의 원인이 되며, 추후 spark가 이러한 문제점을 해결하기 위해 메모리 단에서 연산을 수행하는 방식을 제안합니다.



#### 참고하면 좋은 유튜브

[https://www.youtube.com/watch?v=2RPVFhxps_s&ab_channel=%EC%9D%B4%EA%B0%90TV](https://www.youtube.com/watch?v=2RPVFhxps_s&ab_channel=%EC%9D%B4%EA%B0%90TV)

### Reference

https://data-flair.training/blogs/hadoop-combiner-tutorial/

http://geekdirt.com/blog/map-reduce-in-detail/

https://hadoop.apache.org/docs/r2.8.3/api/org/apache/hadoop/mapreduce/Partitioner.html

https://cloud.google.com/blog/products/gcp/in-memory-query-execution-in-google-bigquery

출처

[https://yeomko.tistory.com/31](https://yeomko.tistory.com/31)