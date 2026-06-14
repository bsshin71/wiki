# Elasticsearch ISM Policy Update (동시성 제어)

- **카테고리**: #elasticsearch #인덱스구성
- **태그**: #elasticsearch #ISM #policy #seq_no #primary_term
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-policy update하기 - BigDataTeam]]

## 1. 핵심 요약

ISM Policy 수정은 **낙관적 동시성 제어**로, 먼저 기존 policy의 `_seq_no`·`_primary_term`을 조회한 뒤 `PUT ...?if_seq_no=&if_primary_term=`로 업데이트합니다.
이 값이 일치하지 않으면(다른 변경 발생) 업데이트가 거부됩니다.

---

## 2. #1 기존 policy의 seq_no / primary_term 확인

```json
GET _opendistro/_ism/policies/roll_over_policy
{
  "_id": "roll_over_policy",
  "_seq_no": 422510,        // ← seq_no
  "_primary_term": 670,     // ← primary_term
  "policy": { ... }
}
```

## 3. #2 if_seq_no & if_primary_term으로 update

```json
PUT _opendistro/_ism/policies/roll_over_policy?if_seq_no=422509&if_primary_term=670
{
  "policy": {
    "default_state": "hot",
    "states": [
      { "name": "hot",
        "actions": [ { "rollover": { "min_doc_count": 2 } } ],
        "transitions": [ { "state_name": "warm" } ] },
      { "name": "warm",
        "actions": [ { "replica_count": { "number_of_replicas": 1 } } ],
        "transitions": [] }
    ],
    "ism_template": [ { "index_patterns": ["test-index*"], "priority": 0 } ]
  }
}
```

> `if_seq_no`/`if_primary_term`이 현재 값과 다르면 409 충돌 → #1을 다시 조회 후 재시도.
> ⚠️ Policy 변경은 기존 관리 인덱스에 즉시 반영되지 않을 수 있어 재적용(change policy)이 필요할 수 있음.

## 4. 연관 개념

- [[2026-06-14-ES10_(Elasticsearch-Hot-Warm-Delete-ISM-Rollover-구성)]]
- [[2026-06-14-ES09_(Elasticsearch-인덱스-구성-방안-ISM-Policy)]]
- [[2026-06-14-ES14_(Elasticsearch-ISM-History-비활성화-및-정리)]]
