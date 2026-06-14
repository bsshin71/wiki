# Elasticsearch Like 검색 (Wildcard·Ngram)

- **카테고리**: #elasticsearch #검색
- **태그**: #elasticsearch #FTS #like #wildcard #ngram #tokenizer
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Like 검색하기 - BigDataTeam]]

## 1. 핵심 요약

ES는 Tokenizer가 문장을 term으로 분리해 **역인덱스(inverted index)** 를 만들고, Like 검색은 이 token 검색이라 **token에 없는 부분 문자열은 `match`로 검색되지 않습니다**.
부분 검색은 **wildcard 쿼리**(token 대상 패턴매칭)나 **ngram analyzer**(token을 더 작게 분리)로 해결합니다.

---

## 2. Full Text Search 동작과 한계

```
GET /index/_analyze
{ "field": "scrpath", "text": "TouchButton:Trade>confirm>TouchButton" }
# standard tokenizer → ["touchbutton:trade", "confirm", "touchbutton"]
```
- `match: {scrpath: "TouchButton"}` → token에 있음 → **성공**
- `match: {scrpath: "Touch"}` → token에 없음 → **실패(0 hits)**

## 3. 해결 1 — Wildcard 쿼리

```json
GET /index/_search
{ "query": { "bool": { "filter": [ { "wildcard": {"scrpath": "*Touch*"} } ] } } }
```
- token 대상 패턴매칭(`*keyword*`).
- **한계**: token에 없는 단어는 여전히 불가 + 성능 부담(전체 token 스캔).

## 4. 해결 2 — Ngram Analyzer (권장)

> token을 min~max 길이로 더 잘게 분리해 부분 문자열도 token에 포함시킴.

```json
PUT my_custom_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_customer_ngram_analyzer": { "type": "custom", "tokenizer": "my_ngram_tokenizer", ... }
      },
      "tokenizer": {
        "my_ngram_tokenizer": { "type": "ngram", "min_gram": 2, "max_gram": 3 }
      }
    }
  }
}
```
- 필드를 `text` + `fields.ngram`(ngram analyzer)으로 매핑 후 `match: {field.ngram: "키워드"}` 검색.
- `min_gram`/`max_gram` = 분리 길이. 작을수록 검색 범위↑·인덱스 크기↑.

## 5. 연관 개념

- [[2026-06-14-ES16_(Elasticsearch-Nested-Object-Array-조회)]]
- [[2026-06-14-ES18_(Elasticsearch-인덱스-생성-데이터-추가-조회-기본)]]
