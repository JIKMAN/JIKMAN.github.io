---
title:  "[Kafka] 도커 컴포즈로 카프카 설치하기"

categories:
  - Kafka
tags:
  - [kafka, message queue, pub-sub]
toc: true
toc_sticky: true
---

> ###  Kafka install

1. `/home/user` 경로에 `touch kafka-docker-compose.yml` 파일을 만들어 아래 yaml 파일을 생성하고 다음 내용을 추가한 후 저장한다.

```bash
version: "3"
services:
  zookeeper:
    image: zookeeper
    restart: always
    container_name: zookeeper
    hostname: zookeeper
    ports:
      - 2181:2181
    environment:
      ZOO_MY_ID: 1
  kafka:
    image: wurstmeister/kafka
    container_name: kafka
    ports:
    - 9092:9092
    environment:
      KAFKA_ADVERTISED_HOST_NAME: [machine-ip-address]
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
  kafka_manager:
    image: hlebalbau/kafka-manager:stable
    container_name: kafka-manager
    restart: always
    ports:
      - "9000:9000"
    environment:
      ZK_HOSTS: "zookeeper:2181"
      APPLICATION_SECRET: "random-secret"
    command: -Dpidfile.path=/dev/null
```

* 외부 서버로 이용한다면 Port를 개방
  * `firewall-cmd --zone=public --permanent --add-por=9092/tcp`
  * `firewall-cmd --reload`



2. `docker-compose -f kafka-docker-compose.yml up -d` 명령어로 해당 yml을 기반으로 구성한 컨테이너를 생성

   * `-d` 커멘드를 추가하면 백그라운드에서 실행

     ![image-20210720144023061](https://github.com/JIKMAN/Kafka-ELK/raw/master/img/image-20210720144023061.png)

3. bash를 통해 컨테이너 접근

   * `sudo docker exec -it kafka /bin/bash`

   * 컨테이너는 `/opt/kafka/`경로에있으므로 접근하여 `bin`경로로 이동

   * `cd /opt/kafka/bin/`

     * ```bash
       # 도커가 꺼져있을 경우 sudo service docker start : 도커 실행 명령어
       ```

---

> ### Topic 생성

메시지를 보내기 위한 test 토픽을 생성

`./kafka-topics.sh --create --bootstrap-server localhost:9092 --topic test`

---

> ###  Producer & Consumer

2개의 터미널을 열어 각각 프로듀서와 컨슈머가 작동하는 지 확인해보자

* 터미널 1
  * `./kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test`
* 터미널 2
  * `./kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning`	

producer에서 입력한 메시지들이 Consumer에서 잘 출력 되는 것을 확인

![image-20210720161654298](https://github.com/JIKMAN/Kafka-ELK/raw/master/img/image-20210720161654298.png)
---

> ### Kafka Manager

kafka manager가 정상 작동되면 9000번 포트로 접근이 가능하다.

![image-20210720162333069](https://github.com/JIKMAN/Kafka-ELK/raw/master/img/image-20210720162333069.png)

