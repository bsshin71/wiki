# Elasticsearch Index 관련 명령 (매핑·Field 삭제·Rollover)

- **카테고리**: #elasticsearch #admin
- **태그**: #elasticsearch #index #mapping #reindex #rollover
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-index 관련 명령 - BigDataTeam]]

## 1. 핵심 요약

인덱스 리스트·매핑(스키마) 조회와 함께, **특정 Field 삭제는 reindex(임시 인덱스 이동)→update_by_query(remove)→재생성** 절차로 수행합니다.
ES는 매핑에서 필드를 직접 제거할 수 없어 reindex 우회가 필요하며, rollover는 `_rollover`로 강제 실행할 수 있습니다.

---

## 2. 인덱스 리스트 / 매핑 조회

```
GET _cat/indices?v                       # health·status·docs·store 등
GET bos.useractlog-*?pretty              # mapping + settings
GET bos.useractlog-202408-000001/_mapping  # mapping만
```

## 3. 특정 Field 삭제 (reindex 우회)

```
# 1) 임시 인덱스로 데이터 이동
POST _reindex
{ "conflicts":"proceed",
  "source":{"index":["bos.useractlog-202408-000001"]},
  "dest":{"index":"bos.useractlog_tmp"} }

# 2) 임시 인덱스에서 Field 삭제
POST bos.useractlog_tmp/_update_by_query
{ "script":{"inline":"ctx._source.remove('client_id')"},
  "query":{"bool":{"must":{"exists":{"field":"client_id"}}}} }

# 3) 원본 삭제 → 4) template에서도 Field 제거 → 5) 임시→원본 reindex(재생성)
DELETE bos.useractlog-202408-000001
PUT _index_template/ism_bosuserlog_rollover { "index_patterns":["bos.useractlog-*"], ... }
POST _reindex { "source":{"index":["bos.useractlog_tmp"]}, "dest":{"index":"bos.useractlog-202408-000001"} }
```

> 핵심: 매핑 필드 직접 제거 불가 → **임시 인덱스 reindex + update_by_query remove + 재생성**.

## 4. Rollover 강제 실행

```
GET _alias/bos.useractlog                       # 현재 writable index 확인
POST bos.useractlog/_rollover?dry_run=false
{ "conditions": {} }                            # 조건 비우면 강제 rollover
```

## 5. 연관 개념

- [[2026-06-14-ES07_(Elasticsearch-Index-상태-진단-조회)]]
- [[2026-06-14-ES08_(Elasticsearch-Index-Rename-reindex)]]
- [[2026-06-14-ES09_(Elasticsearch-인덱스-구성-방안-ISM-Policy)]]
