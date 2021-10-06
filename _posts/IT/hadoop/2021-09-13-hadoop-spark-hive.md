---
title:  "[Hadoop] Hadoop, hive, Spark"

categories:
  - Hadoop
tags:
  - [Spark, hadoop]
toc: true
toc_sticky: true
---

과연 빅데이터 처리 = 하둡(Hadoop)일까?

❓ Hadoop?

하둡은 대용량 데이터를 분산 처리할 수 있는 자바 기반의 오픈소스 프레임워크이다. HDFS(Hadoop Distributed File System) 분산형 파일 시스템을 기반으로 만들어졌다. 데이터 처리 시 HDFS와 분산처리 기술인 MapReduce 데이터셋 병렬 처리 방식에 의해 동작한다.


HDFS
- 대규모 데이터를 분산, 저장, 관리하기 위한 분산 파일 시스템으로서, 저장하고자 하는 파일을 블록 단위로 나누어 분산된 서버에 저장된다.
- 배치작업에 적합하도록 설계되어 있고, 빠른 처리보다 높은 데이터 처리량에 중점을 둔다.
MapReduce
- 대용량 데이터를 분산 처리하기 위한 목적으로 개발된 프로그래밍 모델이자 소프트웨어 프레임워크이다.
- 맵리듀스 프로그래밍 모델은 사용자가 Map과 Reduce 두 함수를 이용하여 데이터를 처리하도록 한다.


⭐ Hive

Hive는 하둡 에코시스템 중에서 데이터를 모델링하고 프로세싱하는 경우 가장 많이 사용하는 데이터 웨어하우징용 솔루션이다

MapReduce는 코드가 복잡해서 짜기가 어렵다. Query로 MapReduce 대부분의 기능을 표현할 수 있다. SQL과 유사한 HiveQL로 데이터를 조회하는 등 MapReduce와 같이 처리할 수 있도록 하는 것이 Hive이다.

Hive는 HDFS등에 있는 파일을 읽어들여 쿼리로 분석을 수행한다. HiveQL을 작성하면 MapReduce로 변환되어 실행되게 된다.

Hive에서 만든 각 테이블의 정보는 Hive 메타 스토어라 불리는 특별한 DB에 저장된다.


Hive는 데이터베이스가 아닌 데이터 처리를 위한 배치 처리 구조이다.



⚡ Spark

실시간성 데이터에 대한 니즈가 증가하며 하둡으로 처리하기에는 속도 측면에서 부적합한 시나리오들이 등장하기 시작하였다.

Spark는 분산 시스템을 사용한 프로그래밍 환경으로 대량의 메모리를 활용하여 고속화를 실현하는 것이다. 인메모리상에서 동작하기 때문에,
반복적인 처리가 필요한 작업에서 속도가 하둡보다 최소 1000배 이상 빠르다.

최근에는 Hadoop + Spark의 연계가 하나의 공식이 되었다. 하둡의 YARN 위에 스파크를 얹고, 실시간성이 필요한 데이터는 스파크로 처리하는 방식으로 아키텍처를 구성하여 동작한다.

Spark는 Hadoop이 아니라 MapReduce를 대체한다. Spark 상의 데이터 처리는 스크립트 언어를 사용할 수 있다 (자바, 스칼라, 파이썬, R 등)


Ref
https://wikidocs.net/book/2203
https://artist-developer.tistory.com/7
https://www.itworld.co.kr/news/100370