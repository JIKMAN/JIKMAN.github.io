---
title:  "[ELK] elastic search config 설정"

categories:
  - ELK
tags:
  - [elk, elastic search]
toc: true
toc_sticky: true
---

### 1. **elasticsearch 폴더 구성 확인**

```bash
[elastic@elastic1 ~]$ cd elasticsearch-7.5.1/
[elastic@elastic1 elasticsearch-7.5.1]$ ls -al
total 556
drwxr-xr-x.  9 elastic elastic    154 Dec 16 18:01 .
drwx------. 16 elastic elastic   4096 Jan 13 18:31 ..
-rw-r--r--.  1 elastic elastic  13675 Dec 16 17:54 LICENSE.txt
-rw-r--r--.  1 elastic elastic 523209 Dec 16 18:01 NOTICE.txt
-rw-r--r--.  1 elastic elastic   8499 Dec 16 17:54 README.textile
drwxr-xr-x.  2 elastic elastic   4096 Dec 16 18:01 bin
drwxr-xr-x.  2 elastic elastic    148 Dec 16 18:01 config
drwxr-xr-x.  9 elastic elastic    107 Dec 16 18:01 jdk
drwxr-xr-x.  3 elastic elastic   4096 Dec 16 18:01 lib
drwxr-xr-x.  2 elastic elastic      6 Dec 16 18:01 logs
drwxr-xr-x. 38 elastic elastic   4096 Dec 16 18:01 modules
drwxr-xr-x.  2 elastic elastic      6 Dec 16 18:01 plugins
[elastic@elastic1 elasticsearch-7.5.1]$ 
```

 

- bin/ : elasticsearch의 실행 파일들이 모여 있는 폴더
- config/ : 각종 환경설정 파일들이 모여 있는 폴더
- jdk/ : Open JDK 폴더. 기존 버전에서는 jdk를 직접 설치해야 했지만 7점대 버전부터였나? 이렇게 내장되어 있다.
- lib/ : elasticsearch 구동에 필요한 라이브러리들이 모여 있는 폴더
- logs/ : 로그 폴더
- modules/ : elastcisearch의 모듈들이 설치되어 있는 폴더
- plugins/ : elasticsearch 플러그인들이 설치되는 폴더

 

### **2. config 파일 구성 확인**

 

```bash
[elastic@elastic1 elasticsearch-7.5.1]$ cd config/
[elastic@elastic1 config]$ ls -al
total 36
drwxr-xr-x. 2 elastic elastic   148 Dec 16 18:01 .
drwxr-xr-x. 9 elastic elastic   154 Dec 16 18:01 ..
-rw-rw----. 1 elastic elastic  2831 Dec 16 17:54 elasticsearch.yml
-rw-rw----. 1 elastic elastic  2204 Dec 16 17:54 jvm.options
-rw-rw----. 1 elastic elastic 17545 Dec 16 18:01 log4j2.properties
-rw-rw----. 1 elastic elastic   473 Dec 16 18:01 role_mapping.yml
-rw-rw----. 1 elastic elastic   197 Dec 16 18:01 roles.yml
-rw-rw----. 1 elastic elastic     0 Dec 16 18:01 users
-rw-rw----. 1 elastic elastic     0 Dec 16 18:01 users_roles
[elastic@elastic1 config]$ 
```

 

- elasticsearch.yml : 제일 중요한 컨피그 파일. elasticsearch의 기본 설정 파일이다.
- jvm.options : elasticsearch를 구동시키기 위한 jvm에 대한 설정 파일. jvm의 heap 메모리 설정이 메인이다.
- log4j2.properties : elasticsearch가 로거로 사용하는 log4j의 설정 파일이다. 로깅에 대한 디테일을 설정할 수 있다.
- role_mapping.yml, roles.yml, users, users_roles : elasticsearch의 사용자 및 권한에 관한 설정 파일들이다.

 

 

### **3. jvm.options 파일 설정 (jvm heap 메모리 설정)**

elasticsearch는 jvm 위에서 동작하며 많은 메모리를 사용하게 된다. jvm의 힙 메모리 영역을 미리 선점해 다른 프로세스가 사용하지 못하도록 하는데 이때 선점해둘 메모리의 용량을 설정하는 내용이다. (다른 설정은 안건드릴 거다.)

 

```bash
[elastic@elastic1 config]$ vi jvm.options 
# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space

-Xms1g
-Xmx1g

################################################################
```

 

- Xms1g : 힙 메모리의 최소 용량을 1GB로 설정
- Xmx1g : 힙 메모리의 최대 용량을 1GB로 설정 (보통 최소/최대 용량을 같게 설정한다)

 

 

### **4. elasticsearch.yml 설정**

제일 중요한 elasticsearch의 기본 설정 파일이다. node 이름, IP 정도만 설정해도 기본 동작이 가능하다.

```bash
[elastic@elastic1 config]$ vi elasticsearch.yml 
node.name: node-1
network.host: 192.168.100.11
cluster.initial_master_nodes: ["node-1"]
```

 

- node.name : elasticsearch의 노드 이름
- network.host : 현재 노드의 IP 주소
- cluster.initial_master_nodes : 현재 cluster의 master node 후보군들이다. 



### **5.  elasticsearch 실행**

```
[elastic@elastic1 config]$ cd ../bin/
[elastic@elastic1 bin]$ ./elasticsearch
```



### **6. error문제**



```bash
ERROR: [2] bootstrap checks failed
[1]: max file descriptors [4096] for elasticsearch process is too low, increase to at least [65535]
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

 

2개의 에러가 확인되는데 첫번째는 max file descriptor 값이 너무 낮다, 

두번째는 vm.max_map_count 값이 너무 낮다는 내용이다.

max file descriptors 값은 하나의 프로세스가 열어볼 수 있는 최대 파일 개수를 제한하는 값이고 vm.max_map_count는 하나의 프로세스가 가질 수 있는 메모리 맵의 최대 개수이다. elasticsearch는 파일 읽기도 잦고 메모리맵도 많이 사용하니 충분히 늘려주자.



먼저 max file descriptors 값을 수정해보자.

root 권한으로 limits.conf 파일에 아래와 같이 추가해주자.

```bash
[elastic@elastic1 bin]$ su -
Password: 
[root@elastic1 ~]# vi /etc/security/limits.conf 
elastic  -  nofile  65535
```

맨 앞의 elastic은 일반계정 이름이다. 즉, elasticsearch를 구동할 계정 이름을 써주면 된다.

  

vm.max_map_count 값은 /etc/sysctl.conf 파일에 아래와 같이 추가해주면 된다.

```bash
[elastic@elastic1 bin]$ su -
Password: 
[root@elastic1 ~]# vi /etc/sysctl.conf 
vm.max_map_count=262144
```


### 7. **방화벽 해제**

방화벽 문제로 실행되지 않으면 방화벽을 중지한다.

```bash
[elastic@elastic1 ~]$ service firewalld stop
```


### 8. **노드 설정**

**1. node-1 설정**

 

```
[elastic@elastic1 elasticsearch-7.5.1]$ vi config/elasticsearch.yml 
cluster.name: elastic-cluster
node.name: node-1
network.host: 192.168.100.11
discovery.seed_hosts: ["192.168.100.11", "192.168.100.12", "192.168.100.13"]
cluster.initial_master_nodes: ["node-1"]

node.master: true
node.data: false
node.ingest: false
node.ml: false
```

 

- cluster.name : 클러스터 이름이다. 이 이름이 같아야 동일 cluster로 인식할 수 있다.
- node.name : 노드 이름
- network.host : 노드의 IP
- discovery.seed_hosts : cluster를 구성할 node의 리스트라 보면 된다. 자기 자신 및 다른 node들의 IP를 적어준다.
- cluster.initial_master_nodes : master 후보 노드의 리스트다. 
- node.master, node,data, node.ingest, node.ml은 node의 기능을 나타내는데 master, data, ingest 및 machine learning이다.
- node-1은 마스터 노드이므로 master만 true, 나머지는 false로 설정

 

 

**2. node-2 설정**

node-2는 아래와 같이 설정하자. 

```
[elastic@elastic2 elasticsearch-7.5.1]$ vi config/elasticsearch.yml 
cluster.name: elastic-cluster
node.name: node-2
network.host: 192.168.100.12
discovery.seed_hosts: ["192.168.100.11", "192.168.100.12", "192.168.100.13"]
cluster.initial_master_nodes: ["node-1"]

node.master: false
node.data: true
node.ingest: false
node.ml: false
```

 

- node-2, node-3은 데이터 노드이니 node.master는 false, node.data는 true로 설정

 

 

**3. node-3 설정**

```
[elastic@elastic3 elasticsearch-7.5.1]$ vi config/elasticsearch.yml 
cluster.name: elastic-cluster
node.name: node-3
network.host: 192.168.100.13
discovery.seed_hosts: ["192.168.100.11", "192.168.100.12", "192.168.100.13"]
cluster.initial_master_nodes: ["node-1"]

node.master: false
node.data: true
node.ingest: false
node.ml: false
```



### 9. **클러스터 상태 확인**

```bash
$ curl http://192.168.100.11:9200/_cluster/health

# 또는

$ curl http://192.168.100.11:9200/_cluster/health?pretty
{
  "cluster_name" : "elastic-cluster",
  "status" : "green",
  "timed_out" : false,
  "number_of_nodes" : 3,
  "number_of_data_nodes" : 2,
  "active_primary_shards" : 0,
  "active_shards" : 0,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 0,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 100.0
}

```



### 10. **샤드 개수 설정**

샤드의 개수는 인덱스를 처음 생성할 때 지정할 수 있다. 프라이머리 샤드 수는 인덱스를 처음 생성할 때 지정하며, **인덱스를 재색인 하지 않는 이상 바꿀 수 없다**. 복제본의 개수는 나중에 변경이 가능하다.

```bash
#프라이머리 샤드 5, 복제본 1 인 books 인덱스 생성

$ curl -XPUT "http://localhost:9200/books" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "number_of_shards": 5,
    "number_of_replicas": 1
  }
}'
```

```bash
# books 인덱스의 복제본 개수를 0 으로 변경

$ curl -XPUT "http://localhost:9200/books/_settings" -H 'Content-Type: application/json' -d'
{
  "number_of_replicas": 0
}'
```

