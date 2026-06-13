# Kafka Topic / Producer / Consumer 명령어

- **카테고리**: #kafka #command
- **태그**: #kafka #command #topic #producer #consumer #partition
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-kafka command - BigDataTeam]]

## 1. 핵심 요약

`kafka-topics`로 토픽 생성/조회/파티션 변경, `kafka-console-producer`/`kafka-console-consumer`로 메시지 push/pull을 수행합니다.
**파티션은 늘리기만 가능(줄이기 불가)** 하며, 동일 메시지를 여러 consumer가 받게 하려면 **group id를 다르게** 지정합니다.

---

## 2. Topic 관리

```bash
# 생성 (replication-factor 3, partitions 1)
kafka-topics --create --zookeeper kafka01:2181 --replication-factor 3 --partitions 1 --topic topictest

# 조회 / 상세
kafka-topics --list --zookeeper kafka01:2181
kafka-topics --describe --zookeeper kafka01:2181 --topic topictest

# 파티션 변경 (⚠️ 늘리기만 가능, 줄이기 불가)
kafka-topics --alter --zookeeper kafka01:2181 --topic topictest --partitions 3
```

## 3. Producer / Consumer

```bash
# push
kafka-console-producer --broker-list kafka01:9092 --topic topictest

# pull (처음부터)
kafka-console-consumer --bootstrap-server kafka01:9092 --topic topictest --from-beginning
```

## 4. 메시지 반복 처리 (다중 consumer)

> 한 메시지를 2개 consumer가 각각 소비하려면 **group id를 다르게** 지정(같은 group이면 분산 소비).

```bash
kafka-console-consumer --bootstrap-server kafka01:9092 --group group1 --topic topictest
kafka-console-consumer --bootstrap-server kafka01:9092 --group group2 --topic topictest
```

## 5. 연관 개념

- [[2026-06-14-K09_(Kafka-기동-순서-모듈)]]
- [[2026-06-14-K05_(Kafka-Connect-Sink-Connector-시작-offset-제어)]]
