---
title:  "[ELK] elastic search 의 데이터 처리"

categories:
  - ELK
tags:
  - [elk, elastic search]
toc: true
toc_sticky: true
---

# Elastic Search 데이터 처리

> ### Rest API
>
> Elasticsearch는 http 프로토콜로 접근이 가능한 REST API를 지원합니다. 자원별로 고유 URL로 접근이 가능하며 http 메서드 PUT, POST, GET, DELETE 를 이용해서 자원을 처리합니다. 이런 특성을 가진 시스템을 보통 **RESTFul** 한 시스템이라고 말합니다.



* #### curl 명령어를 통한 REST API

```bash
# GET 메서드로 elasticsearch 클러스터 조회

$ curl "localhost:9200"

{
  "name" : "596526604b77",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "NkTqQNV8TNWCPQci3IKn1g",
  "version" : {
    "number" : "7.14.0",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "dd5a0a2acaa2045ff9624f3729fc8a6f40835aa1",
    "build_date" : "2021-07-29T20:49:32.864135063Z",
    "build_snapshot" : false,
    "lucene_version" : "8.9.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```



## CURD(Create, Update, Read, Delete)

ElasticSearch에서는 단일 도큐먼트별로 고유한 URL을 갖는다. 도큐먼트에 접근하는 URL은

`http://<호스트>:<포트>/<인덱스>/_doc/<도큐먼트 id> `

```bash
# my_index/_doc/doc_id 에 도큐먼트 입력

$ curl -XPUT "http://localhost:9200/my_index/_doc/doc_id" -H 'Content-Type: application/json' -d'
{
  "name": "Jungik Kim",
  "message": "안녕하세요 Elasticsearch"
}'
{"_index":"my_index","_type":"_doc","_id":"1","_version":1,"result":"created","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":0,"_primary_term":1}
```



> Kibana Dev Tools 활용

### 입력(PUT)

```bash
my_index 인덱스에 도큐먼트 id가 1인 데이터를 입력

PUT my_index/_doc/1
{
  "name":"Jongmin Kim",
  "message":"안녕하세요 Elasticsearch"
}
```

```bash
my_index/_doc/1 최초 입력 결과
{
  "_index" : "my_index",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",  # 처음 입력시 `create`로 표시됨 다른 내용 입력시 `update`로 표시됨
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

* 실수로 도큐먼트가 덮어씌워지는 것을 방지하기 위해 새로입력할 경우에 `_create`를 사용하여 입력하기도 함.

```sql
PUT my_index/_create/1  # `create`를 이용 시 이미 데이터가 있는 경우 error(입력불가능)
{
  "name":"Jongmin Kim",
  "message":"안녕하세요 Elasticsearch"
}
```


### 조회(GET)

```sql
# covid19-time/_doc/"YNG2GXsBtoZ6eMuEM37P" 조회

GET covid19-time/_doc/"YNG2GXsBtoZ6eMuEM37P"

# 조회 결과
{
  "_index" : "covid19-time",
  "_type" : "_doc",
  "_id" : "\"YNG2GXsBtoZ6eMuEM37P\"",
  "_version" : 4,
  "_seq_no" : 156243,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "Confirmed" : 192163
  }
}

```



### 삭제(DELETE)

```sql
# my_index/_doc/1 도큐먼트 삭제

DELETE my_index/_doc/1

# 결과
{
  "_index" : "my_index",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 3,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}

# 삭제된 도큐먼트 조회 시 `false` 응답을 받음
```

```sql
# my_index 인덱스 전체 삭제

DELETE my_index

# 결과
{
  "acknowledged" : true
}

# 삭제된 my_index 인덱스 조회시
{
  "error" : {
    "root_cause" : [
      {
        "type" : "index_not_found_exception",
        "reason" : "no such index [my_index]",
        "resource.type" : "index_expression",
        "resource.id" : "my_index",
        "index_uuid" : "_na_",
        "index" : "my_index"
      }
    ],
    "type" : "index_not_found_exception",
    "reason" : "no such index [my_index]",
    "resource.type" : "index_expression",
    "resource.id" : "my_index",
    "index_uuid" : "_na_",
    "index" : "my_index"
  },
  "status" : 404
}
```



### 수정(POST)

```sql
# POST 명령으로 my_index/_doc 도큐먼트 입력

POST my_index/_doc
{
  "name":"Jungik Kim",
  "message":"안녕하세요 Elasticsearch"
}

# 결과 : 임의의 도큐먼트id 자동 생성됨, PUT으로는 불가능
{
  "_index" : "my_index",
  "_type" : "_doc",
  "_id" : "ZuFv12wBspWtEG13dOut",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

* #### `_update`

입력된 도큐먼트를 수정하기 위해서는 기존 도큐먼트의 URL에 변경될 내용의 도큐먼트 내용을 다시 PUT 하는 것으로 대치가 가능합니다. 

하지만 필드가 여럿 있는 도큐먼트에서 필드 하나만 바꾸기 위해 전체 도큐먼트 내용을 매번 다시 입력하는 것은 번거로운 작업일 것입니다. 

이 때는 `POST <인덱스>/_update/<도큐먼트 id>` 명령을 이용해 원하는 필드의 내용만 업데이트가 가능합니다. 업데이트 할 내용에 "doc" 이라는 지정자를 사용합니다.

  _update API를 이용해서 `my_index/_doc/1` 도큐먼트의 "message" 필드 값을 *"안녕하세요 Kibana"* 로 업데이트를 한 뒤 도큐먼트 내용을 확인 해 보겠습니다. 

`my_index/_doc/1` 도큐먼트를 삭제 하였다면 위의 PUT 내용을 참고해서 새로 입력 한 뒤 아래 명령을 실행합니다.

```sql
#POST 명령으로 my_index/_doc 도큐먼트 입력

POST my_index/_doc
{
  "name":"Jongmin Kim",
  "message":"안녕하세요 Elasticsearch"
}

#my_index/_update/1 도큐먼트의 message 필드 업데이트 실행 결과
{
  "_index" : "my_index",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

```sql
GET /my_index/_doc/1

#message 필드 업데이트 후 도큐먼트 확인 결과
{
  "_index" : "my_index",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 2,  # version이 2로 변경됨
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "name" : "Jongmin Kim",
    "message" : "안녕하세요 Kibana"
  }
}
```

