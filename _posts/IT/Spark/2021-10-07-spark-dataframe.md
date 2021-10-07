---
title:  "[Spark] DataFrame이란?"

categories:
  - Spark
tags:
  - [spark, dataframe]
toc: true
toc_sticky: true
---

## Spark DataFrame의 등장

스파크의 DataFrame은 **행과 열로 구성된 데이터 분산 컬렉션** 입니다.
쉽게 말하면, 더 나은 최적화 기술을 가지고 있는, 관계형 데이터 베이스 구조라고 할 수 있겠죠.

그렇다면, RDD가 있었음에도 불구하고 Dataframe이라는 새로운 데이터 구조 API가 등장한 걸까요?
문제는 **RDD의 성능적 이슈**에 있었습니다.

![img](https://blog.kakaocdn.net/dn/b9B43p/btqI5RMLGZT/QkN43rDcNYxeTBs4GrbZFK/img.png)  RDD의 실행 흐름



\- RDD는 메모리나 디스크에 저장 공간이 충분치 않으면 제대로 동작하지 않습니다.
\- RDD는 스키마(데이터베이스 구조) 개념이 별도로 없습니다.
 구조화된 데이터와 비구조화 데이터를 함께 저장하여 효율성이 떨어집니다.
\- RDD는 기본적으로 **직렬화**(데이터를 배포하거나 디스크에 데이터를 기록할 때마다 JAVA 직렬화 사용)와 **Garbage Collection**(사용하지 않는 객체를 자동으로 메모리에서 해제)을 사용합니다. 이는 메모리 오버헤드를 증가시킵니다.
\- RDD는 별도의 내장된 최적화(Optimize) 엔진이 없습니다. 사용자는 각 RDD를 직접 최적화 해야 합니다.

등등의 이유로 인하여, 스파크 진영에서는 RDD의 한계를 극복하기 위해 DataFrame이라는 개념을 도입합니다.

 

## **DataFrame의 특징**

![img](https://blog.kakaocdn.net/dn/CTt0A/btqI5zrIPkR/DPmAkYSDqQ1iiOITVPswLK/img.png)



\- **구조화된(structed) 데이터 구조** : 앞서 언급한 것처럼, DataFrame은 구조화된 데이터를 다루기 쉽게 하기 위해 만들어진 데이터 구조 입니다. 이를 통해, 스파크 사용자는 SparkSQL 등을 통해 구조화된 데이터의 쿼리를 처리할 수 있습니다.

\- **GC(Garbeage Collection) 오버헤드 감소** : RDD는 데이터를 메모리에 저장합지만, DataFrame은 데이터를 오프-힙(gc의 영향을 받지 않는, 디스크가 아닌 RAM 영역) 영역에 저장합니다. 이를 통해 Garbage Collection의 오버헤드를 감소시켰습니다.

\- **직렬화 오버헤드 감소** : DataFrame은 오프-힙 메모리를 사용한 직렬화를 통하여 오버헤드를 크게 감소시켰습니다.

> **[ Parallel GC ]**
>
> Parallel GC는 Throughput GC로도 알려져 있으며, 기본적인 처리 과정은 Serial GC와 동일하다. 하지만 Parallel GC는 여러 개의 쓰레드를 통해 Parallel하게 GC를 수행함으로써 GC의 오버헤드를 상당히 줄여준다. Parallel GC는 멀티 프로세서 또는 멀티 쓰레드 머신에서 중간 규모부터 대규모의 데이터를 처리하는 애플리케이션을 위해 고안되었으며, 옵션을 통해 애플리케이션의 최대 지연 시간 또는 GC를 수행할 쓰레드의 갯수 등을 설정해줄 수 있다.

\- **Flexibility & Scalability** : DataFrame은 CSV, 카산드라(Cassandra) 등 다양한 형태의 데이터를 직접 지원합니다. 사용자 입장에서는 이를 통한 효율성이 매우 큽니다.

