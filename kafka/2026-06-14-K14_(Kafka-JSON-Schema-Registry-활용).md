# Kafka JSON Schema (Schema Registry 활용)

- **카테고리**: #kafka #config
- **태그**: #kafka #schema-registry #jsonschema #producer #consumer
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-json schema - BigDataTeam]]

## 1. 핵심 요약

Schema Registry(8081)에 JSON Schema를 등록해 producer/consumer/sink가 공통 스키마를 공유합니다.
스키마는 REST API(`/subjects/<topic>-value/versions`)로 등록·조회하며, `kafka-json-schema-console-producer/consumer`로 검증된 메시지를 송수신합니다. array type 선언에 주의합니다.

---

## 2. 스키마 등록/조회 (schema.sh)

```bash
schemaregistry="http://10.1.5.11:8081"
# create: schema를 JSON으로 감싸 POST
curl -X POST -H "Content-Type: application/vnd.schemaregistry.v1+json" \
  --data @$tmpfile ${schemaregistry}/subjects/test-jsonschema-value/versions
# list / desc / id / delete
curl -X GET ${schemaregistry}/subjects/ | jq
curl -X GET ${schemaregistry}/subjects/<subject>/versions/latest/schema | jq
curl -X DELETE ${schemaregistry}/subjects/<subject>
```

## 3. JSON Schema 파일 (array 주의)

```json
{
  "title": "BOS_USER_ACTION_LOG", "type": "object",
  "required": ["udate","loginid","eventkind","task","service"],
  "properties": {
    "udate": {"type":"string","format":"date-time"},
    "loginid": {"type":"string"},
    "eventmessage": {"type":"array","items":{"type":"string"}}
  }
}
```
> 배열은 `"type":"array","items":{...}` 형태로 선언.

## 4. JSON Schema producer / consumer

```bash
# Producer
$CONFLUENT_HOME/bin/kafka-json-schema-console-producer \
  --bootstrap-server $KAFKA:$PORT \
  --property schema.registry.url="http://$SR_HOST:8081" --topic $topic <<END
{"udate":"...","loginid":"newformat",...,"eventmessage":["m1","m2"]}
END

# Consumer
$CONFLUENT_HOME/bin/kafka-json-schema-console-consumer \
  --bootstrap-server $KAFKA:$PORT \
  --property schema.registry.url="http://$SR_HOST:8081" --topic $topic --from-beginning
```

## 5. JSON Schema + ES Sink 연계

> `test-jsonschema` 토픽 수신 시 Schema Registry에서 스키마를 읽어 ES로 sink.

```json
"value.converter": "io.confluent.connect.json.JsonSchemaConverter",
"value.converter.schema.registry.url": "http://10.1.5.11:8081"
```

## 6. 연관 개념

- [[2026-06-14-K11_(Kafka-Elasticsearch-Sink-Connector)]]
- [[2026-06-14-K02_(Kafka-Connect-설정-Debezium-CDC)]]
- [[2026-06-14-K09_(Kafka-기동-순서-모듈)]]
