# Elasticsearch Nested / Object Array 조회

- **카테고리**: #elasticsearch #검색
- **태그**: #elasticsearch #nested #object_array #DSL #조회
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-object array 조회 - BigDataTeam]], [[2026-06-14-nested query 와 Non-nested query 사용 - BigDataTeam]]

## 1. 핵심 요약

object array(예 `eventmessage:[{message,value}...]`)를 일반 object로 매핑하면 **필드 간 pair 결합이 깨져** 잘못된 매칭이 발생합니다.
정확한 array 검색을 위해 매핑을 **`"type": "nested"`** 로 지정하고, 조회 시 **`nested` 쿼리(`path` + 내부 `query`)** 를 사용합니다.

---

## 2. object array의 제약

```json
"eventmessage": [
  { "message": "newevent", "value": "100%" },
  { "message": "zoomin",   "value": "120%" }
]
```
- 일반 object 매핑: 내부 필드가 평탄화되어 `message=zoomin AND value=100%`가 잘못 매칭됨(pair 분리).
- **nested 매핑**: 각 array 요소를 독립 문서로 색인 → pair 결합 유지.

## 3. nested 매핑

```json
"eventmessage": {
  "type": "nested",          // ← 핵심
  "properties": {
    "message": { "type": "text", "fields": { "keyword": { "type": "keyword", "ignore_above": 256 } } },
    "value":   { "type": "text", "fields": { "keyword": { "type": "keyword" } } }
  }
}
```

## 4. nested 쿼리

```json
# 단일 조건
GET newformat/_search
{ "query": { "nested": { "path": "eventmessage",
    "query": { "match": { "eventmessage.message": "unique" } } } } }

# 복수 조건 (pair 결합 검색)
GET newformat/_search
{ "query": { "nested": { "path": "eventmessage",
    "query": { "bool": { "must": [
      { "match": { "eventmessage.message": "unique" } },
      { "match": { "eventmessage.value": "200%" } }
    ] } } } } }
```
> OpenDistro SQL: `SELECT * FROM newformat b, b.eventmessage e WHERE e.message='unique' AND e.value='200%'`

## 5. 복합 쿼리에서 nested 결합 (실전 예)

```json
GET /bos.useractlog-*/_search
{ "query": { "bool": {
    "filter": [
      { "range": { "udate": { "gte": "2021-01-01T00:00:00Z", "lte": "2021-12-27T00:04:11Z" } } },
      { "match": { "scrpath.ngram": "button" } },
      { "nested": { "path": "evmsg",
          "query": { "bool": { "must": { "match": { "evmsg.m.ngram": "symbol" } } } } } }
    ],
    "must": [ { "match": { "cocd": "C001" } } ]
  } },
  "size": 100, "sort": [ { "udate": { "order": "asc" } } ] }
```
> `bool.filter`(스코어 무시, 캐시) + `nested`(array) + `ngram`(부분검색)을 조합.

## 6. 연관 개념

- [[2026-06-14-ES15_(Elasticsearch-Like-검색-Wildcard-Ngram)]]
- [[2026-06-14-ES09_(Elasticsearch-인덱스-구성-방안-ISM-Policy)]]
- [[2026-06-14-ES18_(Elasticsearch-인덱스-생성-데이터-추가-조회-기본)]]
