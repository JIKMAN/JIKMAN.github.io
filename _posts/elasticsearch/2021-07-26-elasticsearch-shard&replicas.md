---
title:  "[ELK] elastic search의 Shard, Replicas"

categories:
  - ELK
tags:
  - [elk, elastic search, shard, replicas]
toc: true
toc_sticky: true
---

# Shard & Replica in ES 


## RAID 0 / RAID 1

 RAID는 여러개의 하드디스크를 용도에 맞게 구성해 사용하는 기술이다. 저장 속도를 높이기 위해 분할 저장을 하거나 안정성을 위해 복제본을 만들거나 패리티 비트를 쓰거나 하는 등의 기술이다.

*  **RAID0**은 stripe라고도 한다. 2개의 disk가 있을 경우 데이터를 나눠서 기록하는 방식이다. 이 방식은 데이터를 저장하는 속도를 높여주는 역할을 한다. disk가 한개라면 동시에 1개 데이터만 기록할 수 있지만 disk가 2개일 경우 동시에 2개 데이터를 분산해서 기록할 수 있다 보면된다.



![img](https://blog.kakaocdn.net/dn/AFgY3/btqA2bUDA7l/GpAGI4fg6DELJgLhx3IrF0/img.png)

* **RAID1**은 mirror라고 한다. 2개의 disk가 있을 경우 같은 데이터를 각 디스크에 똑같이 기록하는 방식이다. 일종의 백업이라 볼 수 있겠다. disk 한개가 고장나도 다른 disk에 데이터가 있기 때문에 안정성이 있다.



![img](https://blog.kakaocdn.net/dn/cyxw3d/btqA6pwQnCm/RbpyY0Y71nsMGsyLiG6WX1/img.png)

 

------

## Shard

 

**shard**는 쉽게 말하면 RAID0과 같다. elasticsearch는 indexing을 할 때 node 내부에 논리적으로 데이터 저장 공간을 만든다. 그리고 그 논리적 공간들에 데이터를 쭉 쓴다. 저장 공간이 1개라면 한번에 1개 데이터만 쓸 수 있겠지만 저장 공간을 논리적으로 나누게 되면 동시에 여러개 데이터를 분산해서 저장할 수 있게 된다. 

인덱스를 생성할 때 별도의 설정을 하지 않으면 **7.0** 버전부터는 **디폴트로 1**개의 샤드로 인덱스가 구성되며 **6.x** 이하 버전에서는 **5개**로 구성됩니다. 클러스터에 노드를 추가하게 되면 샤드들이 각 노드들로 분산되고 디폴트로 1개의 복제본을 생성합니다. 5개의 경우 하나의 index(RDB에서 database와 유사한 개념)를 생성하고 데이터를 indexing하게 되면 node 내부에 논리적으로 5개의 shard가 생성되게 된다. 물론 또 다른 index를 생성하게 되면 그에 대한 shard는 새로 생성된다.

만약 data node가 2개일 경우 아래와 같이 2개의 node에 3개, 2개의 shard가 생성되게 된다.

 

![img](https://blog.kakaocdn.net/dn/eytJKx/btqA7p4eBv9/HH4lR1y8bK2eYI6m2vJXbK/img.png)

 

------

## Replica

 

 **replica**는 RAID1과 유사하다. 데이터가 shard에 저장되게 되면 이 shard에 대한 복제본을 만드는 것이다. 원본 shard가 장애가 나더라도 복제본 shard를 통해 데이터를 제공할 수 있게 되어 가용성을 증가시킬 수 있게 된다. 또한 하나의 데이터를 2개의 shard가 갖고 있기 때문에 검색 성능 향상에도 도움이 된다.

 elasticsearch의 replica 기본 세팅값은 1이다. 즉, 하나의 index를 생성하고 데이터를 indexing을 하게 되면 1개의 replica가 추가로 생성된다는 의미이다. 2개의 data node를 사용하고 shard, replica를 기본 설정으로 사용하게 되면 아래와 같이 shard와 replica가 구성된다. 붉은색 박스가 primary shard(원본 데이터)이고 녹색 박스가 replica shard(복제본)이다. replic의 기본 기능이 복제본을 통해 원본 shard의 장애를 대응하는 것이라 했다. 한 node 안에 동일 shard에 대한 primary와 replica가 함께 존재하는 경우에 해당 node가 장애가 나게 되면 primary, replica shard 모두 죽으므로 정상적으로 데이터를 제공할 수 없게 된다. 따라서 동일 shard에 대한 primary, replica는 동일 node안에 존재할 수 없다. 그래서 아래처럼 node1이 갖고 있는 shard에 대한 replica는 node2에 생성되고 node2가 갖고 있는 shard에 대한 replica는 node1에 생성되는 것이다.



```
노드가 1개만 있는 경우 프라이머리 샤드만 존재하고 복제본은 생성되지 않습니다. Elasticsearch 는 아무리 작은 클러스터라도 데이터 가용성과 무결성을 위해 최소 3개의 노드로 구성 할 것을 권장하고 있습니다.
```



![img](https://camo.githubusercontent.com/90e9550d7a10b09227ff8cf53afb724735c5894730f000b20df225f214ebf6d1/68747470733a2f2f67626c6f627363646e2e676974626f6f6b2e636f6d2f6173736574732532462d4c6e30344461595a61446a6469525f5a734b6f2532462d4c6e55784c6851744c5f684a775979644d5f352532462d4c6e4b686d783331536e4c444e4d4b725f5954253246696d6167652e706e673f616c743d6d6564696126746f6b656e3d66633939613834642d356639662d346433642d616164362d323165353033353637623333)



* 출처 : https://jiseok-woo.tistory.com/8?category=367315

