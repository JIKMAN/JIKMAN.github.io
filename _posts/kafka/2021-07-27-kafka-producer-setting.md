---
title:  "[Kafka] 카프카 프로듀서 setting"

categories:
  - Kafka
tags:
  - [kafka, kafka producer]
toc: true
toc_sticky: true
---

## 카프카 프로듀서 설정

**1. 프로듀서 설정을 분석하는 이유**

프로듀서의 정의를 사전에서 찾아보면 '생산자, 제작자'로 나온다.

카프카에서 프로듀서는 말 그래도 데이터를 생산하는 역할을 한다.

 

프로듀서의 설정값들은 데이터를 브로커에 발송할 때, 발송하는 데이터의 양/ 주기 및 데이터를 받는 브로커와의 네트웍 연결 등을 조절하는데 사용한다. 

앞으로 볼 설정들을 주의깊게 봐야하는 이유는 프로듀서가 비동기로 브로커에 데이터를 발송하기 때문이다.

프로듀서 코드에서 ProducerRecord를 생성해서 send() 메서드로 보낼 때, 바로 데이터가 브로커로 발송되지 않고 **비동기로 데이터를 전송**한다.

그래서 실제로 브로커에 데이터를 발송하기 전까지 데이터를 모아둘 버퍼가 필요하며, 얼마만큼 모아서 보내고, 보낸 데이터의 성공을 어떻게 체크할 지 등의 로직이 설정값에 녹여져 있다.

  

**2. 프로듀서 아키텍처**

Accumulator(축적자), Sender(발송자) 2개의 컴포넌트를 주요하게 살펴보자.

 

다음 클래스의 주석에 다음과 같은 설명이 있다.

- RecordAccumulator
  - 큐 역할을 하는 클래스
  - 한계가 있는 메모리 양을 사용하며 메모리를 다 사용했을 때 블록한다.
- Sender
  - 카프카 브로커에 레코드를 보내는 백그라운드 스레드

아래는 프로듀서에서 브로커로 레코드 발송할 때의 구조를 그린 그림인데 Accumulator와 Sender가 포함되어 있다.

그림에 각 컴포넌트에서 사용하는 설정들을 같이 명시했다. 그림에 설정값이 어디에 표시되어 있는지 참고하면서 각 설정의 의미를 살펴보자. 



![img](https://blog.kakaocdn.net/dn/kISAy/btqEShJFMMM/MYqkvRmOjl8yeHc4CGkMqk/img.png)

* `cat opt/kafka/config/producer.properties`

```bash
# list of brokers used for bootstrapping knowledge about the rest of the cluster
# format: host1:port1,host2:port2 ...
bootstrap.servers=localhost:9092

# specify the compression codec for all data generated: none, gzip, snappy, lz4, zstd
compression.type=none

# name of the partitioner class for partitioning events; default partition spreads data randomly
partitioner.class=

# the maximum amount of time the client will wait for the response of a request
request.timeout.ms=

# how long `KafkaProducer.send` and `KafkaProducer.partitionsFor` will block for
max.block.ms=

# the producer will wait for up to the given delay to allow other records to be sent so that the sends can be batched together
linger.ms=

# the maximum size of a request in bytes
max.request.size=

# the default batch size in bytes when batching multiple records sent to a partition
batch.size=

# the total bytes of memory the producer can use to buffer records waiting to be sent to the server
buffer.memory=
```



**3. Accumulator**

Accumulator는 레코드를 축적하는 역할이다.

그래서 레코드를 축적할 때 사용하는 버퍼의 크기 (buffer.memory), 레코드를 묶는 양 (batch.size), 버퍼가 찾을 때 블록할 시간 (max.block.ms) 설정이 있다.

 

- buffer.memory
  - 프로듀서가 브로커로 데이터를 보내기 위해 잠시 대기하는 메모리량(bytes, default: 33554432, 32MB). 프로듀서가 사용하는 전체 메모리 크기는 buffer.memory와 대략 비슷하다. 정확히 일치하지 않는 것은 압축하는 과정과 브로커에 보내는 과정에서 사용하는 메모리가 추가로 들기 때문이다.
  - 브로커로 레코드를 전달하는 것보다 쌓이는 양이 더 많다면 버퍼가 부족할 수 있다. 버퍼가 부족할 때, max.block.ms 설정이 필요하다.
- max.block.ms
  - 버퍼가 가득 찾을 때 send(), partitionsFor() 메서드를 블록하는 시간이다.
- batch.size
  - 프로듀서는 레코드를 한번에 하나씩 발송하지 않고 묶어서 발송한다. 묶어서 발송해야 네트웍 단계가 줄어드며 성능 향상이 도움이 된다.
  - 같은 파티션으로 보내는 여러 데이터를 함께 배치로 보내기 위한 사이즈. 정의된 크기보다 큰 데이터는 배치를 시도하지 않음. 고가용성이 필요할 경우 배치 사이즈를 지정하지 않음 (bytes, default: 16384, 16KB)
  - 그렇다고 배치가 가득찰 때까지 프로듀서가 기다린다는 것은 아니다.

**4. Sender**

Sender는 레코드를 브로커에 전달하는 역할이다. Accumulator가 쌓아놓은 레코드를 비동기로 브로커에 계속 발송한다.

그래서 Sender와 연결된 컴포넌트를 보면 Accumulator와 Kafka(브로커)가 있다. 

- ack (default:1)
  - 0: 프로듀서는 서버로부터 어떠한 ack도 기다리지 않음. 유실율 높으나 높은 처리량
  - 1: 리더는 데이터를 기록, 모든 팔로워는 확인하지 않음
  - -1(또는 all): 모든 ISR 확인. 무손실

 

Accumulator와 연결된 선에 linger.ms 값이 있다.

- linger.ms
  - 배치형태의 메시지를 보내기 전에 추가적인 메시지들을 위해 기다리는 시간을 조정. 0보다 큰 값을 설정하면 지연은 발생하지만 처리량은 좋아짐 (default:0, 지연 없음)
  - 브로커로 발송하기 전에 Accumulator에 축적된 데이터를 꾸물거리며 가져오는 것이다. 이유는 부하 상황에 브로커에 요청 수를 줄이기 위함이다. 다음 메세지를 바로 보내지 않고 딜레이 시간을 주는 것이다.
  - 한 파티션에 batch.size 만큼 레코드가 있으면 linger.ms 값을 무시하고 발송한다.
  - 기본값은 0으로, 대기하지 않는다.

다음은 브로커와 통신할 때 사용하는 설정들이다.

- retries
  - 레코드 발송에 실패할 경우 재시도 하는 회수이다. 기본값이 2147483647 이다.
  - 여기서 의문은 이렇게 큰 재시도 횟수를 한 메세지에 대해서 계속 발송하면 실패할 경우 다른 메세지는 전송 시도도 못하고 버퍼에 데이터가 쌓이는지 여부이다. retries를 다 채우지 않고도 재시도를 멈추는 다른 설정이 있다. delivery.timeout.ms 이다.
  - 일반적으로 재발송 관련해서 retries 보다 delivery.timeout.ms로 제어한다.
- delivery.timeout.ms
  - send() 메서드를 호출하고 성공과 실패를 결정하는 상한시간이다. __브로커로부터 ack을 받기 위해 대기하는 시간__이며 실패 시 재전송에 허용된 시간이다. 복구할 수 없는 오류가 발생하거나 재시도 횟수를 다 소모하면 delivery.timeout.ms 설정 시간 보다 먼저 에러를 낼 수 있다.
  - request.timeout.ms과 linger.ms의 합보다 같거나 커야 한다.
- max.request.size
  - 프로듀서가 보낼 수 있는 최대 메시지 사이즈. (default: 1MB)
  - 거대한 요청을 피하기 위해 프로듀서에 의한 한 배치에 보내는 사이즈를 제한한다. 압축되지 않은 배치의 최대 사이즈.
  - 서버에서는 별도의 배치 사이즈 설정을 가지고 있다.
- max.in.flight.request.per.connection
  - 블록되기 전에 클라이언트가 보내고 받지 못한 요청의 최대 개수. 한번에 브로커와 통신하는 개수로 이해했다.
  - 만약 1보다 높은 값을 설정하면 재발송 과정에서 순서가 변경되는 위험이 발생한다.
- request.timeout.ms
  - 요청 응답에 대한 클라이언트의 최대 대기 시간. 타임아웃 시간 동안 응답을 받지 못하면 요청을 다시 보낸다.
  - 불필요한 재시도를 줄이려면 브로커 설정 replica.lag.time.max.ms 보다 큰 값이어야 한다.
- replica.lag.time.max.ms
  - 브로커 팔로워가 복제를 위한 패치 요청을 하지 않을 경우 ISR에서 제외하는 시간이다.

공식문서 : https://kafka.apache.org/documentation/#producerconfigs