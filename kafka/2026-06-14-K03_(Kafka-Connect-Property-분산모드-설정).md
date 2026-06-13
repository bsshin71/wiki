# Kafka Connect Property (분산 모드 설정)

- **카테고리**: #kafka #connect
- **태그**: #kafka #connect #config #distributed #MSK
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-confluent connect property - BigDataTeam]]

## 1. 핵심 요약

분산 모드 Connect는 `connect-distributed.properties`에 **bootstrap.servers·group.id·converter·내부 토픽(config/offset/status) + plugin.path**를 설정합니다.
내부 토픽의 `replication.factor`가 `min.insync.replicas`보다 작으면 **NOT_ENOUGH_REPLICAS** 오류가 발생하므로 토픽 재생성 + factor 상향으로 해결합니다.

---

## 2. connect-distributed.properties 핵심

```properties
bootstrap.servers=10.1.5.11:9092,10.1.5.12:9092,10.1.5.13:9092
group.id=connect-cluster

key.converter=org.apache.kafka.connect.json.JsonConverter
value.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=false
value.converter.schemas.enable=false

config.storage.topic=connect-configs
offset.storage.topic=connect-offsets
status.storage.topic=connect-statuses
config.storage.replication.factor=1
offset.storage.replication.factor=1
status.storage.replication.factor=1

plugin.path=share/java,/home/bos/confluent/share/confluent-hub-components
connector.client.config.override.policy=All

# 에러 처리
errors.retry.timeout=-1
errors.log.enable=true
errors.log.include.messages=true
```

## 3. server.properties / zookeeper.properties 주요 값

```properties
# server.properties
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
log.retention.hours=168
zookeeper.connect=10.1.5.11:2181,10.1.5.12:2181,10.1.5.13:2181
listeners=PLAINTEXT://:9092
advertised.listeners=PLAINTEXT://10.1.5.11:9092
message.max.bytes=104857600
batch.size=200000
linger.ms=100
compression.type=lz4
acks=1
```
> AWS MSK와 EC2 self-managed confluent 두 환경 모두 동일 구조(차이는 zookeeper/advertised 주소).

## 4. ⚠️ NOT_ENOUGH_REPLICAS 오류 해결

증상(connect.log):
```
WARN [Producer] Got error produce response ... topic-partition connect-configs-0,
  retrying ... Error: NOT_ENOUGH_REPLICAS
```
원인: 내부 토픽(connect-configs/offsets/statuses)의 `min.insync.replicas` > `replication.factor`.

해결:
1. connector stop
2. connect-configs / connect-offsets / connect-statuses 토픽 삭제
3. `connect-distributed.properties`에서 factor 상향:
   ```properties
   config.storage.replication.factor=2
   offset.storage.replication.factor=2
   status.storage.replication.factor=2
   ```
4. connector 재기동

## 5. 연관 개념

- [[2026-06-14-K01_(Kafka-Connect-설치-confluent-hub)]]
- [[2026-06-14-K02_(Kafka-Connect-설정-Debezium-CDC)]]
- [[2026-06-14-K04_(Kafka-Connector-인증서-truststore-등록)]]
