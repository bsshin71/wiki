# PostgreSQL pg_dump 테이블 DDL 추출

- **카테고리**: #DBMS #PostgreSQL #backup
- **태그**: #PostgreSQL #pg_dump #DDL #schema-only #migration
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-pg_dump 명령 - BigDataTeam]]

## 1. 핵심 요약

테이블 생성 DDL(MySQL의 `SHOW CREATE TABLE` 유사)은 **PG15+는 `pg_get_tabledef()`**, **PG14 이하는 `pg_dump -t <table> --schema-only`** 로 추출합니다.

---

## 2. 테이블 DDL 추출

```sql
-- PostgreSQL 15 이상
SELECT pg_get_tabledef('t1'::regclass);
```
```bash
# PostgreSQL 14 이하
pg_dump -d useract -t public.menu_events --schema-only
```

## 3. 출력 예 (파티션 테이블)

```sql
SET statement_timeout = 0;
SELECT pg_catalog.set_config('search_path', '', false);
...
CREATE TABLE public.menu_events (
    id bigint NOT NULL,
    udate timestamp with time zone NOT NULL,
    pid bigint,
    uid character varying(128),
    path public.ltree NOT NULL,
    leaf_key text
)
PARTITION BY RANGE (udate);
```
> `--schema-only`는 데이터 없이 스키마(테이블·인덱스·제약·파티션 정의)만 출력 → 마이그레이션/DDL 확인용.

## 4. 연관 개념

- [[2026-06-14-PG13_(PostgreSQL-Online-파티션-테이블-재구성)]]
- [[2026-06-02_(PostgreSQL-pgBackRest-백업-가이드)]]
