# Kafka S3 Sink Connector

- **카테고리**: #kafka #connect
- **태그**: #kafka #connect #sink #s3 #TimeBasedPartitioner
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-s3 sink connector - BigDataTeam]]

## 1. 핵심 요약

`S3SinkConnector`로 Kafka 토픽을 S3 버킷에 적재하며, **`TimeBasedPartitioner`** 로 날짜별 경로(`year=/month=/day=/hour=`)에 파티셔닝합니다.
`path.format`에 파티션명을 포함하려면 escape가 필요하고, RegexRouter는 null exception을 유발하므로 멀티 토픽→단일 경로 통합은 미지원입니다.

---

## 2. 설정 예

```json
{
  "connector.class": "io.confluent.connect.s3.S3SinkConnector",
  "topics": "test-jsonschema",
  "flush.size": "5",
  "rotate.interval.ms": "6000",
  "timestamp.extractor": "Record",
  "aws.access.key.id": "AKIA...", "aws.secret.access.key": "****",
  "s3.bucket.name": "bos-dev-log-analysis", "s3.region": "ap-northeast-2",
  "s3.part.size": "52428800",
  "storage.class": "io.confluent.connect.s3.storage.S3Storage",
  "format.class": "io.confluent.connect.s3.format.json.JsonFormat",
  "partitioner.class": "io.confluent.connect.storage.partitioner.TimeBasedPartitioner",
  "partition.duration.ms": "600000",
  "path.format": "'\\'_year\\''=YYYY/'\\'_month\\''=MM/'\\'_day\\''=dd/'\\'_hour\\''=HH",
  "locale": "ko_KR", "timezone": "Asia/Seoul"
}
```

## 3. 주의점

- **날짜별 경로에 파티션명 포함 시 `path.format` escape 필요**(위 예시).
- **멀티 토픽을 하나의 path로 write하는 방법 미발견** — S3 Sink에서 `RegexRouter` 사용 시 **null exception 발생**.

## 4. 연관 개념

- [[2026-06-14-K11_(Kafka-Elasticsearch-Sink-Connector)]]
- [[2026-06-14-K13_(Kafka-Camel-SFTP-Sink-Connector-SMT)]]
- [[2026-06-14-K05_(Kafka-Connect-Sink-Connector-시작-offset-제어)]]
