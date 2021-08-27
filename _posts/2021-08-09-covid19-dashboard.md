---
title:  "COVID-19 Dash board Project"
layout: splash
categories:
  - project
tags:
  - [Elastic search, ELK, Kibana, Dashboard]
toc: true
toc_sticky: true
---
# COVID-19 Dash board Project

#### Elastic Stack을 이용한 Covid-19 대시보드 만들기 프로젝트
![dashboard](https://github.com/JIKMAN/data-engineer/raw/master/img/covid19_dashboard.PNG)


## 프로젝트 설계
* 개요
  
  유의미한 데이터를 수집하고 로컬 환경에서 ELK Stack를 활용하여 직접 대시보드로 구현해 보기 위해 프로젝트를 진행하였다.

* 사전 준비
  * 대시보드 생성에 이용할 데이터 수집
  * Docker 이용
  

* 진행 과정
  
  로컬 환경에서 docker를 이용해 Elastic Search 및 Kibana를 설치한다. 수집한 데이터를 구조에 맞게 인덱싱 및 정제 한 후 ES, Kibana를 이용해 Dash board를 생성한다.
    
    
* 피드백

    효과적인 대시보드를 만들기 위해 고려해야할 부분은 생각보다 많았다. 누가 이용하는지, 무슨 목적으로 만들어지는지 등 많은 부분을 신경쓰다 보니 시각화는 생각보다 고민이 많이 필요한 단계인 것 같다.
    
    또한, 데이터의 전처리를 했음에도 시각화를 하다가 잘못된 데이터를 발견하는 경우가 종종 있었는데, 그때그때 업데이트를 해주다보니 생각보다 번거로웠다. 데이터를 꼼꼼하게 전처리하는 과정이 중요한 것 같다.

    나름대로 괜찮은 결과물을 완성했지만 부족한 부분을 많이 느꼈다. 이번 계기로 시각화 툴에 대해서도 좀 더 관심을 가지고 공부해봐야겠다.

* #### docker-compose 이용하여 elasticsearch-kibana server 구성

```yaml
version: "3.3"
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.14.0
    container_name: elasticsearch
    restart: always
    environment:
      - xpack.security.enabled=false
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile:
        soft: 65536
        hard: 65536
    cap_add:
      - IPC_LOCK
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
  kibana:
    container_name: kibana
    image: docker.elastic.co/kibana/kibana:7.14.0
    restart: always
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200    # 엘라스틱 서치 포트를 환경변수로 지정
    ports:
      - 5601:5601
    depends_on:
      - elasticsearch                                   # 엘라스틱 서치 실행 후 키바나가 실행되도록 의존성 설정
    volumes:
      elasticsearch-data:
```



* #### 데이터 소스

데이터 소스를 살펴보면 데이터 row는 총 156239개로 **Date, Country, Confirmed, Recovered, Deaths**로 구성되어 있고, 코로나가 발생한 날로부터 현재 날짜인 21년 8월 3일까지의 확진자 수, 완치자 수, 사망자 수가 나라별로 표기되어 있다. 

데이터 소스 출처 - https://datahub.io/core/covid-19

데이터를 살펴보니 중간에 미국, 영국의 통계 중 완치자 수가 누적되지 않고 0으로 되어있는 부분이 있었다.
확진자 수가 포함된 데이터이기 때문에 데이터를 삭제하기 보다는, 빈 완치자 수를 채워넣기로 했다.

인터넷을 살펴보니 해당 국가의 정확한 완치자 수 통계치가 나와있지 않아서, 평균 증가값으로 대체했다.

그러나 정확한 통계치가 아닌 임의의 값으로 대체한 데이터이기 때문에 데이터의 정확성을 위해 해당국가의 완치자 수는 시각화 자료로 사용을 최대한 자제하는 것이 좋겠다. (특히나 COVID-19와 관련된 통계는 정확성이 무엇보다 중요한 것 같다.)

![preprocessing](https://github.com/JIKMAN/data-engineer/blob/master/img/preprocessing.png?raw=true)

* #### Index Mapping

저용량의 데이터이기 때문에 엘라스틱서치에 자체적으로 업로드하여 인덱스를 생성했다.

엘라스틱에서 자동으로 매핑을 생성해주지만 이번에 사용할 데이터 타입에 long type을 지정할 필요는 없어서 공간을 절약하기 위해 **short**으로 변경해 주었다.

![index-setting](https://github.com/JIKMAN/data-engineer/raw/master/img/index-setting.PNG)



* #### Create index pattern

생성한 Covid-19 Index의 인덱스 패턴을 생성해준다.

날짜 필드는 timestamp로 지정한 후 인덱스 패턴을 생성해주면 데이터 조회가 가능하다.

![search](https://github.com/JIKMAN/data-engineer/raw/master/img/dovid-data.PNG)



* #### Visualization

visualize library를 이용하여, 원하는 데이터를 조회해 시각화 자료를 생성해주었다.

![image-20210827214806735](https://github.com/JIKMAN/data-engineer/raw/master/img/visual.PNG)


* #### Update Data

시각화 자료를 만들다 보니, 국내 2021-07-26 데이터와 2021-07-25 데이터 수치가 동일하게 나타나 있는 오류를 발견하였다. 

필요한 데이터이기 때문에 삭제하지는 않고, 26일의 데이터를 25일과 27일 데이터 값의 평균으로 대체해 주었다.

```sh
# 데이터 수정
# PUT 명령어로도 가능하지만, _update를 이용하면 원하는 필드만 지정해서 변경할 수 있다.

POST covid19-time/_update/QJiVh3sBAmil_3VJbMjF
{
  "doc": {
    "Confirmed" : 191797,
    "Deaths" : 2080,
    "Recovered" : 168930
  }
}
```

![update](https://github.com/JIKMAN/data-engineer/raw/master/img/update.PNG)

다시한번 조회 해보면

```bash
GET covid19-time/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "Date": "2021-07-26"
          }
        },
        {
          "match": {
            "Country/Region": "Korea, South"
          }
        }
      ]
    }
  }
}
```

정상적으로 변경되었음을 확인할 수 있다.

![update-complete](https://github.com/JIKMAN/data-engineer/raw/master/img/update-complete.PNG)


지도 서비스를 이용하기 위해 공식 문서를 찾아본 결과 ISO 3166 code, 위경도 등 다양한 방법으로 지도 표시가 가능했는데, Country 필드의 타입을 **"geo_point"**로 변경해 country name을 이용하여 지도에 표시하였다.

참고 - 공식문서[(https://maps.elastic.co/#file/world_countries)](https://maps.elastic.co/#file/world_countries)

이렇게 해서,

* 현재 날짜 까지의 전세계 감염자 수 및 사망자 수 / 국내 감염자 수, 사망자 수
* 전세계 확진자 수 Top 5 국가
* 국내 및 해외의 일일 감염자 수의 추이 비교
* 나라별 감염/회복/사망 인원을 나타내는 지도

를 나타내는 covid19 대시보드를 완성하였다.

* #### Dashboard

![dashboard](https://github.com/JIKMAN/data-engineer/raw/master/img/covid19_dashboard.PNG)