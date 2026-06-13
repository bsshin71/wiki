# Kafka Elasticsearch Sink Connector

- **카테고리**: #kafka #connect
- **태그**: #kafka #connect #sink #elasticsearch #TimestampRouter
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-elasticsearch  sink connector - BigDataTeam]]

## 1. 핵심 요약

`ElasticsearchSinkConnector`로 Kafka 토픽을 Elasticsearch 인덱스에 적재합니다. `topics.regex`로 멀티 토픽을 읽고, **`TimestampRouter` SMT**로 날짜별 인덱스(`devboslog-yyyyMMdd`)에 통합 저장합니다.
`schema.ignore=true`/`key.ignore=true`로 스키마 없이 색인하며, key.ignore=true 시 document id는 `topic+partition+offset`이 됩니다.

---

## 2. 주요 property

| 항목 | 설명 |
|------|------|
| `connector.class` | `io.confluent.connect.elasticsearch.ElasticsearchSinkConnector` |
| `topics` / `topics.regex` | 읽을 토픽(정규식으로 멀티 토픽) |
| `connection.url` | ES 서버 URL |
| `schema.ignore` | 색인에 스키마 미사용 여부 |
| `key.ignore` | true면 document id = topic+partition+offset |
| `connection.username/password` | ES 접속 계정 |
| `batch.size` / `flush.timeout.ms` | bulk insert 크기·타임아웃 |
| `max.retries` / `read.timeout.ms` | 재시도·읽기 타임아웃 |

## 3. 설정 예 (JSON without schema + 멀티 토픽)

```json
{
  "connector.class": "io.confluent.connect.elasticsearch.ElasticsearchSinkConnector",
  "connection.url": "https://vpc-es-dev-log-....amazonaws.com",
  "value.converter": "org.apache.kafka.connect.json.JsonConverter",
  "value.converter.schemas.enable": "false",
  "schema.ignore": "true", "key.ignore": "true",
  "connection.username": "esuser", "connection.password": "****",
  "topics.regex": "(bos.).*",
  "batch.size": "2000", "flush.timeout.ms": "20000",
  "max.retries": "10", "read.timeout.ms": "120000",
  "transforms": "TimestampRouter",
  "transforms.TimestampRouter.type": "org.apache.kafka.connect.transforms.TimestampRouter",
  "transforms.TimestampRouter.topic.format": "devboslog-${timestamp}",
  "transforms.TimestampRouter.timestamp.format": "yyyyMMdd"
}
```
> 멀티 토픽을 날짜별 단일 인덱스(`devboslog-yyyyMMdd`)로 통합 저장.

## 4. JSON Schema(Schema Registry) 사용 시

```json
"key.converter": "org.apache.kafka.connect.storage.StringConverter",
"value.converter": "io.confluent.connect.json.JsonSchemaConverter",
"value.converter.schema.registry.url": "http://10.1.5.11:8081"
```

## 5. 연관 개념

- [[2026-06-14-K14_(Kafka-JSON-Schema-Registry-활용)]]
- [[2026-06-14-K12_(Kafka-S3-Sink-Connector)]]
- [[2026-06-14-K03_(Kafka-Connect-Property-분산모드-설정)]]
