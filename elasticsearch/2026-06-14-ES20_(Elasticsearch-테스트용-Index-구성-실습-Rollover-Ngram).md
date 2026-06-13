# Elasticsearch 테스트용 Index 구성 실습 (Rollover + Ngram)

- **카테고리**: #elasticsearch #인덱스구성
- **태그**: #elasticsearch #실습 #ISM #rollover #ngram #template
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-테스트용 index 구성과정 - BigDataTeam]]

## 1. 핵심 요약

테스트 인덱스 구성 end-to-end 실습으로, **Date Math seed 인덱스 생성 → ISM rollover policy → index_template(rollover_alias + ngram analyzer) 적용** 순서를 통합합니다.
ISM policy와 template의 `rollover_alias`를 일치시키고, ngram custom analyzer를 매핑에 포함해 부분검색을 지원합니다.

---

## 2. ① seed 인덱스 생성 (Date Math + write alias)

```
# PUT <test-bos.useractlog-{now/d{yyyyMM}}-000001>
PUT %3Ctest-bos.useractlog-%7Bnow%2Fd%7ByyyyMM%7D%7D-000001%3E
{ "aliases": { "test-bos.useractlog": { "is_write_index": true } } }

GET _cat/indices?v      # 생성 확인
```

## 3. ② ISM Rollover Policy 생성

> Dev Tools → Index Management → State management policies 또는 DSL 직접 생성.

```json
PUT _opendistro/_ism/policies/test_hot_delete_rollover_policy
{
  "policy": {
    "default_state": "hot",
    "states": [
      { "name": "hot",
        "actions": [
          { "replica_count": { "number_of_replicas": 1 } },
          { "rollover": { "min_size": "2mb", "min_index_age": "1d" } }
        ],
        "transitions": [ { "state_name": "delete", "conditions": { "min_index_age": "10d" } } ] },
      { "name": "delete", "actions": [ { "delete": {} } ], "transitions": [] }
    ],
    "ism_template": [ { "index_patterns": ["test-bos.useractlog-*"], "priority": 0 } ]
  }
}
```

## 4. ③ index_template (rollover_alias + ngram analyzer)

```json
GET _cat/templates                              # 기존 template 확인
PUT _index_template/test_ism_bosuserlog
{
  "index_patterns": ["test-bos.useractlog-*"],
  "template": {
    "settings": { "index": {
      "opendistro": { "index_state_management": { "rollover_alias": "test-bos.useractlog" } },
      "analysis": { "filter": { ... }, "analyzer": { "my_customer_ngram_analyzer": { ... } },
                    "tokenizer": { "ngram": { "type": "ngram", "min_gram": ..., "max_gram": ... } } }
    } },
    "mappings": { "properties": { /* text + fields.ngram */ } }
  }
}
```

## 5. 핵심 체크포인트

- ISM policy `ism_template`의 index_patterns와 template의 patterns 일치.
- policy의 `rollover` action과 template settings의 `rollover_alias` 정합성.
- seed 인덱스명 `-000001` + write alias로 rollover 연쇄 생성.
- ngram analyzer를 template에 포함해 신규 rollover 인덱스에도 자동 적용.

## 6. 연관 개념

- [[2026-06-14-ES10_(Elasticsearch-Hot-Warm-Delete-ISM-Rollover-구성)]]
- [[2026-06-14-ES13_(Elasticsearch-Index명-날짜함수-Date-Math)]]
- [[2026-06-14-ES15_(Elasticsearch-Like-검색-Wildcard-Ngram)]]
