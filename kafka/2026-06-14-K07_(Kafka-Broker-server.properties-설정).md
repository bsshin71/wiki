# Kafka Broker server.properties 설정

- **카테고리**: #kafka #config
- **태그**: #kafka #config #broker #listeners #retention
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-kafka property - BigDataTeam]]

## 1. 핵심 요약

Broker `server.properties`의 핵심은 **`broker.id`(브로커별 고유)**, **`listeners`/`advertised.listeners`(접속 주소)**, **log.dirs·retention(보존)**, **내부 토픽 replication factor**입니다.
운영(비개발) 환경에서는 내부 토픽 replication factor를 **3 이상** 권장합니다.

---

## 2. Server Basics / Socket

```properties
delete.topic.enable=true
broker.id=1                                  # 브로커별 고유 정수

listeners=PLAINTEXT://kafka01:9092           # 수신 주소
advertised.listeners=PLAINTEXT://10.1.5.11:9092  # producer/consumer에 알릴 주소

num.network.threads=3
num.io.threads=8                             # disk I/O 포함 요청 처리 스레드
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600           # OOM 방지
```

## 3. Log Basics / Retention

```properties
log.dirs=/home/bos/confluent/data/kafka-logs
num.partitions=1                             # 토픽 기본 파티션 수(병렬성)
num.recovery.threads.per.data.dir=1

log.retention.hours=168                      # 7일 보존
log.segment.bytes=1073741824                 # 세그먼트 최대 크기
log.retention.check.interval.ms=300000
#log.retention.bytes=...                      # 크기 기반 보존(시간과 OR 조건)
```

## 4. 내부 토픽 / Zookeeper / Rebalance

```properties
# __consumer_offsets, __transaction_state 복제 계수
offsets.topic.replication.factor=1           # 운영은 3 권장
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

zookeeper.connect=127.0.0.1:2181
zookeeper.connection.timeout.ms=6000

group.initial.rebalance.delay.ms=0           # 개발 0, 운영은 3000(불필요 rebalance 방지)
```

> ⚠️ `offsets.topic.replication.factor`는 토픽 최초 생성 시점에만 적용 — 운영은 처음부터 3으로 설정.

## 5. 연관 개념

- [[2026-06-14-K08_(Kafka-Zookeeper-properties-설정)]]
- [[2026-06-14-K06_(Kafka-설치-Apache-Confluent-Debezium-CDC)]]
- [[2026-06-14-K03_(Kafka-Connect-Property-분산모드-설정)]]
