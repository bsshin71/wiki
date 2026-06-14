# Kafka Connect 설정 (Debezium MySQL CDC)

- **카테고리**: #kafka #connect
- **태그**: #kafka #connect #debezium #CDC #source #sink
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-connect 설정 - BigDataTeam]]

## 1. 핵심 요약

Debezium MySQL Connector로 **source(CDC 캡처)**·**sink(적재)** 커넥터를 REST API(`POST /connectors`)로 등록합니다.
MySQL→MySQL sink 시 Debezium이 timestamp를 ISO-8601로 변환해 MySQL이 인식 못하므로 **custom UTCTimestampConverter**가 필요하고, `transforms.unwrap`(ExtractNewRecordState)로 Debezium envelope를 평탄화합니다.

---

## 2. 커넥터 등록 스크립트 (REST)

```bash
NAME=connector_name; HOST=localhost; PORT=8083
# CONTEXT_STR 배열의 key=value를 JSON config로 조립해 POST
curl -i -X POST -H "Content-Type:application/json" $HOST:$PORT/connectors/ -d '{ "name":"...", "config":{ ... } }'
# 삭제
curl -X DELETE http://$HOST:$PORT/connectors/$NAME
```

## 3. Source 커넥터 (MySQL CDC)

```json
{
  "connector.class": "io.debezium.connector.mysql.MySqlConnector",
  "database.hostname": "10.1.5.11", "database.port": "3306",
  "database.user": "test", "database.password": "test",
  "database.server.id": "9999", "database.server.name": "mysql",
  "database.include.list": "test",
  "database.history.kafka.bootstrap.servers": "kafka01:9092",
  "database.history.kafka.topic": "mysql-bos-schema",
  "snapshot.mode": "when_needed",
  "decimal.handling.mode": "string",
  "transforms": "unwrap",
  "transforms.unwrap.type": "io.debezium.transforms.ExtractNewRecordState",
  "transforms.unwrap.delete.handling.mode": "none"
}
```

## 4. Sink 커넥터 (MySQL, ⚠️ timestamp 변환)

> source MySQL의 timestamp가 ISO-8601로 변환되어 sink MySQL이 인식 못함 → **custom converter 필요**.
> https://github.com/aminkr/UTCTimestampConverter

```json
{
  "table.include.list": "DBBOS.T1,DBBOS.T2,...",
  "transforms": "route,unwrap",
  "transforms.route.type": "org.apache.kafka.connect.transforms.RegexRouter",
  "transforms.route.regex": "([^.]+)\\.([^.]+)\\.([^.]+)",
  "transforms.route.replacement": "$3",
  "key.converter": "io.confluent.connect.avro.AvroConverter",
  "value.converter": "io.confluent.connect.avro.AvroConverter",
  "key.converter.schema.registry.url": "http://localhost:8081",
  "converters": "timestampConverter",
  "timestampConverter.type": "snapp.kafka.connect.util.UTCTimestampConverter"
}
```

- `RegexRouter`: `server.db.table` 토픽명을 `table`만 남기도록 변환.
- Avro 사용 시 Schema Registry(8081) 필요.

## 5. 연관 개념

- [[2026-06-14-K01_(Kafka-Connect-설치-confluent-hub)]]
- [[2026-06-14-K03_(Kafka-Connect-Property-분산모드-설정)]]
- [[2026-06-13-02_(MySQL-GTID-기반-복제)]]
- [[2026-06-13-01_(MySQL-Binary-Log-Position-복제)]] — Debezium이 캡처하는 MySQL binlog
- [[2026-06-13-20_(MySQL-Binary-Log-활성화-비활성화)]] — CDC 전제: binlog 활성화
