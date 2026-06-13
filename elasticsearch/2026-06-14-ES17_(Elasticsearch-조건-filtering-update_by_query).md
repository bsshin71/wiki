# Elasticsearch 조건 filtering + update_by_query

- **카테고리**: #elasticsearch #admin
- **태그**: #elasticsearch #update_by_query #painless #read_only #shrink
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-특정조건으로 filtering하고 update하기 - BigDataTeam]]

## 1. 핵심 요약

특정 조건 문서의 필드만 변경하려면 **`_update_by_query`** 에 `bool.filter` 조건과 **painless 스크립트**(`ctx._source.field = ...`)를 사용합니다.
shrink된 인덱스는 read_only로 막혀 있어 `cluster_block_exception(403)`이 발생하므로, **임시 writable 전환 → update → read_only 복귀** 절차가 필요합니다.

---

## 2. 조건 필터 + 필드 update

```json
POST /bos.useractlog*/_update_by_query?conflicts=proceed
{
  "query": { "bool": { "filter": [
    { "term": { "svrtype.keyword": "1" } },
    { "term": { "acc_t": "4" } }
  ] } },
  "script": { "source": "ctx._source.acc_t = '6'", "lang": "painless" }
}
```
> `svrtype=1 AND acc_t=4` 문서의 `acc_t`를 6으로 변경. `conflicts=proceed`로 버전 충돌 무시.

## 3. ⚠️ read_only 인덱스 write 오류 (shrink)

```
"type": "cluster_block_exception",
"reason": "index [shrink-...] blocked by: [FORBIDDEN/8/index write (api)];"  // status 403
```

### 해결: 임시 writable → update → read_only 복귀

```json
# 1) writable 전환
PUT /shrink-...-*/_settings
{ "index.blocks.read_only": false,
  "index.blocks.read_only_allow_delete": null,
  "index.blocks.write": null }

# 2) _update_by_query 수행

# 3) read_only 복귀
PUT /shrink-...-*/_settings
{ "index.blocks.read_only": true }
```

## 4. 연관 개념

- [[2026-06-14-ES06_(Elasticsearch-Index-관련-명령-매핑-Field삭제-Rollover)]]
- [[2026-06-14-ES16_(Elasticsearch-Nested-Object-Array-조회)]]
