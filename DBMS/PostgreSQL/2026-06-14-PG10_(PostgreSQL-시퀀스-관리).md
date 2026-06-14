# PostgreSQL 시퀀스 관리

- **카테고리**: #DBMS #PostgreSQL #admin
- **태그**: #PostgreSQL #sequence #nextval #cache #OWNED_BY
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-postgresql 시퀀스 관리 - BigDataTeam]]

## 1. 핵심 요약

시퀀스는 규칙적으로 숫자를 생성하는 single-row 객체로, `nextval/currval/setval/lastval` 함수로 다룹니다.
**CACHE**는 미리 할당하지만 세션 종료 시 미사용분을 잃어 값이 건너뛰어지고, **OWNED BY**는 컬럼 의존성을 부여해 컬럼 삭제 시 함께 삭제되도록 합니다.

---

## 2. CREATE SEQUENCE

```sql
CREATE SEQUENCE seq1;
CREATE SEQUENCE seq1 AS BIGINT INCREMENT BY 2;
CREATE SEQUENCE seq1 MINVALUE -10 MAXVALUE 1 START WITH 1 CACHE 20 CYCLE;
CREATE SEQUENCE seq1 OWNED BY test.t1.i1;
```
| 옵션 | 설명 |
|------|------|
| TEMPORARY | 해당 세션에서만 존재(세션 종료 시 삭제), 동명 우선 |
| CACHE n | n개 미리 할당. **세션 종료 시 미사용분 손실 → 다음 사용 시 skip** |
| OWNED BY t.c | 컬럼 의존성 → 컬럼 삭제 시 시퀀스도 삭제 |
| CYCLE | MAX 도달 시 MIN으로 순환 |

> CACHE 예: `CACHE 10000` 후 nextval=1 → 세션 재접속 시 nextval=10001(미사용분 건너뜀).

## 3. 시퀀스 함수

| 함수 | 설명 |
|------|------|
| `nextval('seq')` | 다음값 |
| `currval('seq')` | 현재값 |
| `lastval()` | 현 세션 마지막 사용 시퀀스 값 |
| `setval('seq', 42)` | 현재값 설정(다음 nextval=43) |
| `setval('seq', 42, false)` | 다음 nextval=42 |

```sql
nextval('myschema.foo')   -- 스키마 명시
SELECT setval('foo', 42, false);  -- 다음 nextval은 42
```

## 4. ALTER / DROP SEQUENCE

```sql
ALTER SEQUENCE IF EXISTS seq1 OWNER TO chlee;
ALTER SEQUENCE IF EXISTS seq1 RENAME TO seq2;
ALTER SEQUENCE IF EXISTS seq1 SET SCHEMA chlee;

DROP SEQUENCE IF EXISTS seq1 CASCADE;   -- 의존 객체 함께 삭제
DROP SEQUENCE IF EXISTS seq1 RESTRICT;  -- 의존 객체 있으면 거부
```

## 5. 의존성 객체 찾기

```sql
SELECT refobjid::regclass AS table_name
FROM pg_depend WHERE objid = 'seq1'::regclass AND deptype = 'a';
```

## 6. 연관 개념

- [[2026-06-14-PG06_(PostgreSQL-스키마-관리-search_path)]]
- [[2026-06-14-PG04_(PostgreSQL-데이터베이스-관리)]]
