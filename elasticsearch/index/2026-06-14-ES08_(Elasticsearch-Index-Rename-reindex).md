# Elasticsearch Index Rename (reindex)

- **카테고리**: #elasticsearch #admin
- **태그**: #elasticsearch #index #rename #reindex
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-index rename 하기 - BigDataTeam]]

## 1. 핵심 요약

ES는 인덱스 직접 rename이 없어 **`_reindex`로 새 이름에 복사 → 원본 삭제** 절차로 이름을 바꿉니다.
여러 인덱스를 하나로 합칠 때도 `_reindex`의 source에 배열로 지정(`conflicts: proceed`)합니다.

---

## 2. 인덱스 이름 변경

```
# 1) 새 이름으로 복사
POST _reindex
{ "source": { "index": "bosuseractlog_test_20211020", "query": {"match_all": {}} },
  "dest":   { "index": "bosuseractlog_test_new" } }

# 2) 원본 삭제
DELETE bosuseractlog_test_20211020
# (필요 시 원래 이름으로 다시 reindex)
```

## 3. 여러 인덱스 합치기

```
POST _reindex
{ "conflicts": "proceed",
  "source": { "index": [
    "bos.useractlog-20220324", "bos.useractlog-20220325", "bos.useractlog-20220328", ...
  ]},
  "dest": { "index": "bosuseractlog_tmp" } }
```
> `conflicts: proceed` — 문서 ID 충돌 시 건너뛰고 계속 진행.

## 4. 연관 개념

- [[2026-06-14-ES06_(Elasticsearch-Index-관련-명령-매핑-Field삭제-Rollover)]]
- [[2026-06-14-ES09_(Elasticsearch-인덱스-구성-방안-ISM-Policy)]]
