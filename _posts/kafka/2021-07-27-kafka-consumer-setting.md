---
title:  "[Kafka] 카프카 컨슈머 setting"

categories:
  - Kafka
tags:
  - [kafka, kafka consumer]
toc: true
toc_sticky: true
---

## Kafka Consumer

New와 Old 컨슈머 차이 : old는 오프셋을 zk에 저장

#### Consumer config

- fetch.min.bytes : 한번에 가져올 수 있는 최소 데이터 사이즈. 만약 지정한 사이즈보다 작은 경우 요청에 응답하지 않고 데이터가 누적될 때까지 기다림 (default: 1)
- auto.offset.reset: 카프카 초기 오프셋이 없거나 현재 오프셋이 더 이상 존재하지 않을 경우(데이터가 삭제)에 다음 옵션으로 리셋함
  - earliest, lastest, none (default: latest)
- fetch.max.bytes : 한 번에 가져올 수 있는 최대 데이터 사이즈 (default: 52428800, 50MB)
- request.timeout.ms : 요청에 대해 응답을 기다리는 최대 시간 (default: 305000)
- session.timeout.ms : 컨슈머와 브로커 사이의 세션 타임 아웃 시간(default: 10초). 타임아웃되면 해당 컨슈머는 종료되거나 장애로 판단하고 리밸런스를 시도함. heartbeat.interval.ms(기본 3초)와 밀접한 관련이 있음. GC를 고려하여 적당한 시간 조정 필요
- max.poll.records: 단일 호출 poll()에 대해 최대 레코드 수를 조정. 이 옵션을 통해 app이 폴링 루프에서 데이터 양을 조정할 수 있음 (default: 500)
- max.poll.interval.ms: 하트비트를 통해 살아는 있으나 실제 메세지를 가져가지 않을 경우. 주기적으로 poll을 호출하지 않으면 장애라고 판단하고 컨슈머 그룹에서 제외 (default: 300,000)
- fetch.max.wait.ms: fetch.min.bytes에 의해 설정된 데이터보다 적은 경우 요청에 응답을 기다리는 최대 시간 (default: 500)