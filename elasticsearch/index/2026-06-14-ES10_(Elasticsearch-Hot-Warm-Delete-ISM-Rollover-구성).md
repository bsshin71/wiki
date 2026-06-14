# Elasticsearch Hot-Warm-Delete ISM + Rollover 구성

- **카테고리**: #elasticsearch #인덱스구성
- **태그**: #elasticsearch #ISM #hot-warm #rollover #lifecycle
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Hot-warm 과 Rollover - BigDataTeam]]

## 1. 핵심 요약

ISM Policy로 **hot(rollover)→warm(warm_migration)→delete** 데이터 라이프사이클을 자동화합니다.
구성 순서는 **① policy 생성 → ② policy·index 매핑 + rollover용 index_template → ③ seed index(`-000001`, write alias) 생성**입니다.

---

## 2. 구성 순서 (AWS OpenSearch / OpenDistro 7.10+)

### #1 ISM Policy 생성 (hot→warm→delete)

```json
{
  "policy": {
    "policy_id": "hot_warm_delete_workflow",
    "default_state": "hot",
    "states": [
      { "name": "hot",
        "actions": [ { "rollover": { "min_index_age": "7d" } } ],
        "transitions": [ { "state_name": "warm", "conditions": { "min_index_age": "10d" } } ] },
      { "name": "warm",
        "actions": [ { "timeout": "24h", "retry": {"count":5,"backoff":"exponential","delay":"1h"},
                       "warm_migration": {} } ],
        "transitions": [ { "state_name": "delete", "conditions": { "min_index_age": "12d" } } ] },
      { "name": "delete", "actions": [ { "delete": {} } ], "transitions": [] }
    ],
    "ism_template": [ { "index_patterns": ["bosactlog*"], "priority": 0 } ]
  }
}
```

### #2 rollover용 index_template (policy·alias 매핑)

```json
PUT _index_template/ism_rollover
{
  "index_patterns": ["bosactlog*"],
  "template": {
    "settings": { "index": {
      "number_of_shards": "2", "number_of_replicas": "1",
      "opendistro": { "index_state_management": { "rollover_alias": "bosactlog" } }
    } },
    "mappings": { "properties": { "udate": { "type": "date", "format": "yyyy-MM-dd'T'HH:mm:ss.SSSZ" } } }
  }
}
```

### #3 seed index 생성 (write alias)

```json
PUT bosactlog-000001
{ "aliases": { "bosactlog": { "is_write_index": true } } }
```
> seed 생성 실패 시 `settings`에 `opendistro.index_state_management.rollover_alias: "bosactlog"` 명시.

## 3. 핵심 포인트

- `rollover_alias`는 template settings와 seed index 양쪽에서 일치해야 함.
- hot의 `rollover` action(min_index_age/min_doc_count/min_size)이 충족되면 `-000002`로 전환.
- warm_migration은 warm 노드(`data_warm`)로 샤드 이동.

## 4. 연관 개념

- [[2026-06-14-ES11_(Elasticsearch-Rollover-Alias-개념)]]
- [[2026-06-14-ES09_(Elasticsearch-인덱스-구성-방안-ISM-Policy)]]
- [[2026-06-14-ES12_(Elasticsearch-ISM-Policy-Update-동시성제어)]]
- [[2026-06-14-ES02_(Elasticsearch-Cluster-구축-아키텍처)]]
