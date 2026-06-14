# PostgreSQL 스키마 관리 (search_path)

- **카테고리**: #DBMS #PostgreSQL #admin
- **태그**: #PostgreSQL #schema #search_path #admin
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-postgresql 스키마 관리 - BigDataTeam]]

## 1. 핵심 요약

스키마는 데이터베이스 하위의 객체 묶음으로, 객체 참조 시 **`search_path`** 순서로 스키마를 탐색합니다(기본 `"$user", public`).
DB 기본 스키마 변경은 `ALTER DATABASE ... SET search_path`(재접속 반영) 또는 `SET search_path`(즉시 반영)로 수행합니다.

---

## 2. 벤더별 용어 비교

| 개념 | PostgreSQL | Oracle | MySQL |
|------|-----------|--------|-------|
| database | database | database | database |
| schema | schema | user | database |
| user/role | user/role(=user) | user/role | user/role(8.0) |

> 스키마 객체: 테이블·인덱스·시퀀스·뷰·PSM 등. 스키마가 다르면 동명 객체 생성 가능.

## 3. CREATE / ALTER / DROP SCHEMA

```sql
CREATE SCHEMA test;                          -- owner = current_user
CREATE SCHEMA AUTHORIZATION test;            -- schema=test, owner=test
CREATE SCHEMA test2 AUTHORIZATION test;

ALTER SCHEMA schema1 RENAME TO schema2;
ALTER SCHEMA schema1 OWNER TO test;

DROP SCHEMA IF EXISTS schema1;               -- 비어있을 때만
DROP SCHEMA IF EXISTS schema1 CASCADE;       -- 속한 객체까지 삭제
DROP SCHEMA IF EXISTS schema1 RESTRICT;      -- 객체 있으면 거부(기본)
```

## 4. search_path

```sql
SHOW search_path;          -- "$user", public
SET search_path TO chlee3; -- 세션 즉시 반영
```
> search_path에 없는 스키마의 객체는 **스키마 명시 없이는 조회 실패**(`relation does not exist`) → `schema.table`로 명시.

## 5. DB 기본 스키마 변경

```sql
ALTER DATABASE dvdrental SET search_path TO test_schema;  -- 재접속해야 반영
SET search_path TO test_schema;                           -- 즉시 반영(세션)
```
> `ALTER DATABASE`는 **재접속(`\c`) 후** 적용, `SET`은 현재 세션 즉시 적용.

## 6. 연관 개념

- [[2026-06-14-PG04_(PostgreSQL-데이터베이스-관리)]]
- [[2026-06-14-PG07_(PostgreSQL-권한-관리-GRANT)]]
- [[2026-06-14-PG09_(PostgreSQL-테이블스페이스-관리)]]
