# Elasticsearch Index명 날짜함수 (Date Math)

- **카테고리**: #elasticsearch #인덱스구성
- **태그**: #elasticsearch #date_math #index #rollover #alias
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-index명에 날짜함수 적용 - BigDataTeam]]

## 1. 핵심 요약

인덱스명에 **Date Math(`<test-index-{now/d{yyyyMMdd}}-000001>`)** 를 적용하면 생성 시점 날짜가 인덱스명에 자동 반영됩니다.
URL에서는 `<`, `{`, `}` 등을 **percent-encoding** 해야 하며, rollover를 위해 인덱스명은 `-000001`로 끝나고 write alias를 지정합니다.

---

## 2. 인덱스 생성 (Date Math + alias)

```
# 논리 표현: PUT <test-index-{now/d{yyyyMMdd}}-000001>
PUT %3Ctest-index-%7Bnow%2Fd%7ByyyyMMdd%7D%7D-000001%3E
{
  "aliases": { "bos-logs": { "is_write_index": true } }
}
```
- alias 명시 + `is_write_index: true`
- 인덱스명 패턴 끝은 **`-000001`**(rollover 시 `-000002`로 증가)

## 3. 데이터 입력 (동일 Date Math URL)

```
PUT %3Ctest-index-%7Bnow%2Fd%7ByyyyMMdd%7D%7D-000001%3E/_doc/1
{ "user": "testuser", "post_date": "2020-05-08T14:12:12", "message": "ISM testing" }
```

## 4. Date Math 문법 / 인코딩

| 논리 | 인코딩 |
|------|--------|
| `<` / `>` | `%3C` / `%3E` |
| `{` / `}` | `%7B` / `%7D` |
| `/` | `%2F` |

- `now/d` = 오늘(일 단위 반올림), `{yyyyMMdd}` = 날짜 포맷.
- 참고: Elasticsearch Date Math index names.

## 5. 연관 개념

- [[2026-06-14-ES11_(Elasticsearch-Rollover-Alias-개념)]]
- [[2026-06-14-ES10_(Elasticsearch-Hot-Warm-Delete-ISM-Rollover-구성)]]
