# Kafka Connect Sink Connector 시작 offset 제어

- **카테고리**: #kafka #connect
- **태그**: #kafka #connect #sink #offset #consumer-group
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-Starting a Kafka Connect sink connector at the end of a topic - BigDataTeam]]

## 1. 핵심 요약

Sink 커넥터를 **토픽 끝(최신 offset)부터** 시작하려면 커넥터 config로 지정하거나, `kafka-consumer-groups`로 해당 consumer group의 offset을 `--to-latest`로 리셋합니다.
Connect sink는 내부적으로 `connect-<커넥터명>` consumer group을 사용합니다.

---

## 2. 두 가지 방법

1. **커넥터 config 설정** — https://rmoff.net/2019/08/09/starting-a-kafka-connect-sink-connector-at-the-end-of-a-topic/
2. **kafka-consumer-groups offset reset** — https://www.letmecompile.com/kafka-consumer-offset-reset/

## 3. 현재 offset/LAG 조회

```bash
# 전체 그룹
kafka-consumer-groups --bootstrap-server 10.1.5.11:9092 --describe --all-groups

# 특정 그룹 (connect-<sink커넥터명>)
kafka-consumer-groups --bootstrap-server 10.1.5.11:9092 --describe \
  --group connect-s3-sink-connector
# GROUP TOPIC PARTITION CURRENT-OFFSET LOG-END-OFFSET LAG ...
```
> sink 커넥터는 활성 멤버가 없을 때(`has no active members`)만 offset reset 가능 → 먼저 커넥터 중지.

## 4. offset을 최신으로 reset

```bash
kafka-consumer-groups --bootstrap-server 10.1.5.11:9092 \
  --group connect-s3-sink-connector --all-topics \
  --reset-offsets --to-latest --execute
```

## 5. AWS MSK에서 실행

```bash
kafka-consumer-groups --bootstrap-server b-1.<msk-cluster>...amazonaws.com:9092 \
  --group connect-elastic-single-sink-connector --all-topics \
  --reset-offsets --to-latest --execute
```

> `--to-latest` 외 `--to-earliest`, `--to-offset N`, `--shift-by N`, `--to-datetime` 등 사용 가능. `--dry-run`으로 먼저 확인 권장.

## 6. 연관 개념

- [[2026-06-14-K02_(Kafka-Connect-설정-Debezium-CDC)]]
- [[2026-06-14-K03_(Kafka-Connect-Property-분산모드-설정)]]
