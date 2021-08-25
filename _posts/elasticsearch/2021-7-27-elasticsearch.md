---
title:  "[ELK] elastic search 란"

categories:
  - ELK
tags:
  - [elk, elastic search]
toc: true
toc_sticky: true
---

# Elastic Search

  Elasticsearch는 Elastic사에서 제공하는 ELK 스택의 하나로, 기본적으로 모든 데이터를 색인하여 저장하고 검색, 집계 등을 수행하며 결과를 클라이언트 또는 다른 프로그램으로 전달하여 동작하게 합니다.

  기존 관계 데이터베이스 시스템에서는 다루기 어려운 전문검색(Full Text Search) 기능과 점수 기반의 다양한 정확도 알고리즘, 실시간 분석 등의 구현이 가능합니다. 또한 다양한 플러그인들을 사용해 손쉽게 기능의 혹장이 가능하며 아마존 웹 서비스(AWS), 마이크로소프트 애저(MS Azure) 같은 클라우드 서비스 그리고 하둡(Hadoop) 플랫폼들과의 연동도 가능합니다.

 

> ## elastic search의 특징

## 오픈소스

  Elasticsearch의 핵심 기능들은 Apache 2.0 라이센스로 배포되고 있고 Elastic Stack의 모든 제품들은 (https://github.com/elastic) 깃헙 리파지토리 에서 소스들을 찾을 수 있습니다. 6.3 버전 부터는 Elastic 라이센스와 Apache 라이센스가 섞여 있지만 각각의 버전에 대해 별도 배포판이 존재하고 license 파일에서 어떤 경로의 파일들이 어떤 라이센스로 되어 있는지 확인이 가능합니다. 현재는 x-pack 디렉토리 아래 있는 파일들만이 Elastic 라이센스를 따르고 그 외의 파일들은 Apache 라이센스를 따릅니다.

## 실시간 분석 (real-time)

  Elasticsearch의 가장 큰 특징 중 하나는 실시간(real-time) 분석 시스템 입니다. 

  Elasticsearch는 하둡 시스템과 달리 Elasticsearch 클러스터가 실행되고 있는 동안에는 계속해서 데이터가 입력 (검색엔진에서는 색인 – indexing 이라고 표현합니다) 되고, 그와 동시에 실시간에 가까운 (near real-time) 속도로 색인된 데이터의 검색, 집계가 가능합니다.

## 검색 엔진

 기본적으로 역파일 색인(inverted file index)라는 구조로 데이터를 저장합니다. JSON 형식으로 데이터를 전달합니다. JSON형식은 간결하고 개발자들이 다루기 편한 구조로 되어 있어 색인 할 대상 문서를 가공 하거나 다른 클라이언트 프로그램과 연동하기에 용이합니다.

  또한 key-value 형식이 아닌 문서 기반으로 되어 있기에 복합적인 정보를 포함하는 형식의 문서를 있는 그대로 저장이 가능하며 사용자가 직관적으로 이해하고 사용할 수 있습니다. Elasticsearch에서 질의에 사용되는 쿼리문이나 쿼리에 대한 결과도 모두 JSON 형식으로 전달되고 리턴됩니다.

  다만 JSON이 Elasticsearch가 지원하는 유일한 형식이기 사전에 입력할 데이터를 JSON 형식으로 가공하는 것이 필요합니다. CSV, Apache log, syslog등과 같이 널리 사용되는 형식들은 Logstash에서 변환을 지원하고 있습니다.

## RESTFul API

  현재 대규모 시스템들은 대부분 마이크로 서비스 아키텍처(MSA)를 기본으로 설계됩니다. 이러한 구조에 빠질 수 없는 것이 REST API와 같은 표준 인터페이스 입니다. Elasticsearch는 Rest API를 기본으로 지원하며 모든 데이터 조회, 입력, 삭제를 http 프로토콜을 통해 Rest API로 처리합니다.

## 멀티테넌시 (multitenancy) 

  Elasticsearch의 데이터들은 인덱스(Index) 라는 논리적인 집합 단위로 구성되며 서로 다른 저장소에 분산되어 저장됩니다. 서로 다른 인덱스들을 별도의 커넥션 없이 하나의 질의로 묶어서 검색하고, 검색 결과들을 하나의 출력으로 도출할 수 있는데, Elasticsearch의 이러한 특징을 멀티테넌시 라고 합니다.

  ## Indexing in Elastic Search

  > __Elasticsearch는 어떻게 작동하나요?__

 *로그, 시스템 메트릭, 웹 애플리케이션 등 다양한 소스로부터 원시 데이터가 Elasticsearch로 흘러들어갑니다. 데이터 수집은 원시 데이터가 Elasticsearch에서 색인되기 전에 구문 분석, 정규화, 강화되는 프로세스입니다. Elasticsearch에서 일단 색인되면, 사용자는 이 데이터에 대해 복잡한 쿼리를 실행하고 집계를 사용해 데이터의 복잡한 요약을 검색할 수 있습니다. Kibana에서 사용자는 데이터를 강력하게 시각화하고, 대시보드를 공유하며, Elastic Stack을 관리할 수 있습니다.*
>
> __Elasticsearch 인덱스는 무엇인가요?__

 *Elasticsearch 인덱스는 서로 관련되어 있는 문서들의 모음입니다. Elasticsearch는 JSON 문서로 데이터를 저장합니다. 각 문서는 일련의 키(필드나 속성의 이름)와 그에 해당하는 값(문자열, 숫자, 부울, 날짜, 값의 배열, 지리적 위치 또는 기타 데이터 유형)을 서로 연결합니다.Elasticsearch는 역 인덱스라고 하는 데이터 구조를 사용하는데, 이것은 아주 빠른 전체 텍스트 검색을 할 수 있도록 설계된 것입니다. 역 인덱스는 문서에 나타나는 모든 고유한 단어의 목록을 만들고, 각 단어가 발생하는 모든 문서를 식별합니다. 색인 프로세스 중에, Elasticsearch는 문서를 저장하고 반전된 인덱스를 구축하여 거의 실시간으로 문서를 검색 가능한 데이터로 만듭니다. 인덱스 API를 사용해 색인이 시작되며, 이를 통해 사용자는 특정한 인덱스에서 JSON 문서를 추가하거나 업데이트할 수 있습니다.*
>
 *출처 : elastic 공식 홈페이지 - https://www.elastic.co/kr/what-is/elasticsearch*

 

 

 

------

## Inverted Index

 elastic 공식 홈페이지의 설명을 보면 **"역 인덱스(inverted index)"**라는 용어가 나온다.  

**index**는 "색인"이라는 뜻이다. 일반적인 RDB를 다뤄본 사람이라면 index의 개념은 어느정도 알고 있을 것이다. index는 RDB에서 쿼리 성능을 향상시키는 기술 중 하나다. indexing이 되지 않은 상태에서 데이터를 조회하게 되면 조회하고자 하는 데이터가 어디에 있는지 모르기 때문에 테이블의 모든 데이터를 다 뒤져보게 된다. 데이터가 총 100개 저장되어 있고 그 중 1개 데이터를 조회할때 최악의 경우엔 100개 데이터를 모두 뒤져봐야 원하는 데이터를 찾을 수 있게 된다는 얘기다.

 index는 쉽게 말해 목차를 만든다고도 얘기들을 하지만 정확히 말하면 B+ Tree 구조로 별도의 index 테이블을 만든다. 따라서 해당 컬럼 값으로 데이터를 찾거나 정렬을 하게 될 경우 인덱싱이 되지 않았을 때보다 더 빠르게 접근이 가능해진다.

 



![img](https://blog.kakaocdn.net/dn/DhB3u/btqA4yVvrlI/3nmCAujgoWGIUc3tK7Ka6k/img.png)



특정 컬럼을 기준으로 정렬해 트리구조를 만들어버리면 검색하는데 훨씬 유리해지게 된다. 이게 인덱싱의 원리이다. 

**inverted index**는 역 인덱스라고 했다. 보통 데이터가 저장되면 1번째 row에 어떤 데이터가 있다는 식으로 저장이 된다. inverted index는 반대로 어떤 데이터는 몇번째 row에 있다는 방식으로 저장한다.

 

![img](https://blog.kakaocdn.net/dn/bFXnCo/btqA56KZnsr/FbKKI996qV2SK43406ABZ0/img.png)

elasticsearch가 강점을 보이는 부부은 문장이나 여러 단어들의 조합들이 저장될 때이다. 문장은 여러 단어들로 구성이 되어있고 그 중 중요한 키워드도 있고 큰 의미가 없는 단어들도 있다. elasticsearch는 데이터를 저장할때 의미있는 단어들을 추출해 해당 단어들로 inverted index를 생성한다. 그렇게 되면 단어를 검색할때 어떤 데이터들에 해당 단어가 포함되어 있는지 빠르게 확인이 가능해진다.

 

![img](https://blog.kakaocdn.net/dn/zAKrL/btqA2OdJ0gN/OWDXAuRa0bzp7gWB5fRkAK/img.png)inverted index 예제

 

위에서 보면 "나는 책을 읽는다" 라는 문장은 "나", "책", "읽다" 세개 키워드가 제일 중요한 키워드이다. elastcisearch는 데이터를 저장하기 전에 analyzer라는 친구가 해당 데이터를 분석하고 tokenizer로 적절히 짤라 키워드를 생성해 inverted index로 저장할 수 있도록 해준다. 예를들어 language analyzer의 경우엔 문장을 분석해 "가다", "간다", "갔다" 등을 "가다"로 통일해 키워드를 생성한다. tokenizer는 분석된 문장을 공백이든 -든 기준을 잡아 키워드 단위로 짤라주는 역할을 한다. elasticsearch는 수많은 analyzer와 tokenizer가 존재하며 이를 잘 활용해야 elasticsearch를 제대로 활용한다 볼 수 있겠다.

  "나는 책을 읽는다."와 같은 문장들이 저장되어 있고 그 중 "영화"라는 단어가 들어간 데이터만 찾고 싶을때 옆의 inverted index 테이블을 보면 누가봐도 한눈에 아주 쉽게 "영화" 라는 단어가 어디에 있는지 확인할 수 있다. 이게 elasticsearch에서 말하는 inverted index이고 elasticsearch가 검색엔진으로써 빠른 검색이 가능한 이유이다.

  출처 : https://blog.kakaocdn.net/