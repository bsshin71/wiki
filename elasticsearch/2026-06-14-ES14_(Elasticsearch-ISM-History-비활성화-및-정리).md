# Elasticsearch ISM History 비활성화 및 정리

- **카테고리**: #elasticsearch #인덱스구성
- **태그**: #elasticsearch #ISM #history #cluster_settings #정리
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-disable ism 변경 history - BigDataTeam]]

## 1. 핵심 요약

ISM template 변경마다 `.opendistro-ism-managed-index-history*` 인덱스가 생겨 조회가 번잡해집니다.
**`opendistro.index_state_management.history.enabled: false`** 로 history 생성을 막고, 이미 쌓인 history는 별도 ISM policy로 일정 기간 후 자동 삭제합니다.

---

## 2. History 생성 비활성화

```json
PUT _cluster/settings
{
  "persistent": {
    "opendistro.index_state_management.history.enabled": false
  }
}
```

## 3. 기존 History 자동 삭제 Policy

```json
{
  "policy": {
    "policy_id": "delete_old_ism_history",
    "default_state": "hot",
    "states": [
      { "name": "hot", "actions": [],
        "transitions": [ { "state_name": "delete", "conditions": { "min_index_age": "1d" } } ] },
      { "name": "delete", "actions": [ { "delete": {} } ], "transitions": [] }
    ],
    "ism_template": [
      { "index_patterns": [".opendistro-ism-managed-index-history*"], "priority": 0 }
    ]
  }
}
```
> history 인덱스가 너무 많으면 위 policy로 1일 경과분을 자동 삭제.

## 4. 연관 개념

- [[2026-06-14-ES12_(Elasticsearch-ISM-Policy-Update-동시성제어)]]
- [[2026-06-14-ES10_(Elasticsearch-Hot-Warm-Delete-ISM-Rollover-구성)]]
- [[2026-06-14-ES07_(Elasticsearch-Index-상태-진단-조회)]]
