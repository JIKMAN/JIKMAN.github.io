---
title:  "[Kafka] 카프카 서버 setting"

categories:
  - Kafka
tags:
  - [kafka, kafka server, message queue, pub-sub]
toc: true
toc_sticky: true
---

## **Kafka server 설정**

>  server.properties

```bash
# Broker의 ID로 Cluster내에서 Broker를 구분하기위해 사용.
broker.id=0
 
# 생성되지 않은 토픽을 자동으로 생성할지 여부. 기본값은 true.
auto.create.topics.enable=false
 
# Broker가 받은 데이터를 관리위한 저장공간.
log.dirs=C:/work/kafka_2.12-2.1.0/data/kafka
 
# Broker가 사용하는 호스트와 포트를 지정, 형식은 PLAINTEXT://your.host.name:port 을 사용
listeners=PLAINTEXT://:9092
 
# Producer와 Consumer가 접근할 호스트와 포트를 지정, 기본값은 listeners를 사용.
advertised.listeners=PLAINTEXT://localhost:9092
 
# 서버가 받을 수 있는 메세지의 최대 크기, 기본값 1MB.
# Consumer에서는 fetch.message.max.bytes를 사용하는데
# message.max.bytes >= fetch.message.max.bytes로 조건에 맞게 잘 설정해야한다.
message.max.bytes=1000000
 
# 네트워크 요청을 처리하는 쓰레드의 수, 기본값 3.
num.network.threads=3
 
# I/O가 생길때 마다 생성되는 쓰레드의 수, 기본값 8.
num.io.threads=8
 
# 서버가 받을 수 있는 최대 요청 사이즈이며, 서버 메모리가 고갈 되는걸 방지함.
# JAVA의 Heap 보다 작게 설정해야 함, 기본값 104857600.
socket.request.max.bytes=104857600
 
# 소켓 서버가 사용하는 송수신 버퍼 (SO_SNDBUF, SO_RCVBUF) 사이즈, 기본값 102400.
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
 
# 토픽당 파티션의 수를 의미하며, 입력한 수만큼 병렬처리를 할 수 있지만 데이터 파일도 그만큼 늘어남. 기본값 1.
num.partitions=1
 
# 세그먼트 파일의 기본 사이즈, 기본값 1GB.
# 토픽별로 수집한 데이터를 보관하는 파일이며, 세그먼트 파일의 용량이 차면 새로운 파일을 생성.
log.segment.bytes=1073741824
 
# 세그먼트 파일의 삭제 주기, 기본값 hours, 168시간( 7일 ).
# 옵션 [ bytes, ms, minutes, hours ] 
log.retention.hours=168
 
# 세그먼트 파일의 삭제 주기에 따른 처리, 기본값은 delete.
# 옵션 [ compact, delete ]
# compact는 파일에서 불필요한 records를 지우는 방식.
log.cleanup.policy=delete
 
# 세그먼트 파일의 삭제 여부를 체크하는 주기, 기본값 5분.
log.retention.check.interval.ms=300000
 
# 세그먼트 파일의 삭제를 처리할 쓰레드의 수. 기본값 1.
log.cleaner.threads=1
 
# 오프셋 커밋 파티션의 수, 한번 배포되면 수정할 수 없음. 기본값 50.
offsets.topic.num.partitions=50
 
# 토픽에 설정된 replication의 인수가 지정한 값보다 크면 새로운 토픽을 생성하고
# 작으면 브로커의 수와 같게 된다. 기본값 3.
offsets.topic.replication.factor=1
 
# 주키퍼의 접속 정보.
zookeeper.connect=localhost:2181
 
# 주키퍼 접속 시도 제한시간.
zookeeper.connection.timeout.ms=6000
```

* 참고 : https://free-strings.blogspot.com/2016/04/blog-post.html
* 공식 문서 : https://kafka.apache.org/documentation/#configuration

