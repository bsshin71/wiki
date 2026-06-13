# Elasticsearch 인덱스 생성·데이터 추가·조회 (기본)

- **카테고리**: #elasticsearch #기본
- **태그**: #elasticsearch #index #mapping #document #search
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-인덱스생성및 데이터추가(simple) - BigDataTeam]]

## 1. 핵심 요약

`PUT /<index>`로 settings(shards/replicas)+mappings를 지정해 인덱스를 만들고, `PUT /<index>/_doc/<id>`로 문서를 색인, `GET /<index>/_search`로 조회합니다.
매핑에 없는 필드를 추가하면(`newAddColumn`) **dynamic mapping**으로 자동 추가되며, text 필드는 기본적으로 `.keyword` 서브필드를 가집니다.

---

## 2. 인덱스 생성 (settings + mappings)

```json
PUT /movie
{
  "settings": { "number_of_shards": 3, "number_of_replicas": 0 },
  "mappings": { "properties": {
    "movieCd": { "type": "integer" },
    "movieNm": { "type": "text" },
    "repNationNm": { "type": "keyword" },
    "regGenreNm": { "type": "keyword" }
  } }
}
```

## 3. 데이터 추가 (색인)

```json
PUT /movie/_doc/1
{ "movieCd": "1", "movieNm": "반지의 제왕3 왕의 귀환", "prdtYear": "2003", "repNationNm": "뉴질랜드" }

# 매핑에 없는 필드 추가 → dynamic mapping 자동 생성
PUT /movie/_doc/2
{ "movieCd": "2", "movieNm": "...", "newAddColumn": "aaaaaaaaa" }
```

## 4. 조회

```json
GET movie/_search
{ "query": { "match_all": {} } }
# hits.total.value, hits[]._source 확인
```

## 5. 매핑 구조 조회

```json
GET /movie/_mapping
# text 필드는 fields.keyword(type keyword, ignore_above 256) 서브필드 자동 생성
```

> `text`(분석/전문검색) vs `keyword`(정확 매칭/정렬/집계). text 필드의 `.keyword`로 정확 매칭 가능.

## 6. 연관 개념

- [[2026-06-14-ES15_(Elasticsearch-Like-검색-Wildcard-Ngram)]]
- [[2026-06-14-ES16_(Elasticsearch-Nested-Object-Array-조회)]]
- [[2026-06-14-ES05_(Elasticsearch-Dev-Tools-DSL-조회)]]
