# PostgreSQL 테이블스페이스 관리

- **카테고리**: #DBMS #PostgreSQL #admin
- **태그**: #PostgreSQL #tablespace #물리저장 #admin
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-postgresql 테이블스페이스 관리 - BigDataTeam]]

## 1. 핵심 요약

테이블스페이스는 데이터베이스 객체의 **물리적 저장 위치**를 추상화·관리하는 개체로, SSD/HDD를 목적별로 분리 구성할 수 있습니다.
⚠️ 테이블스페이스 생성은 **트랜잭션으로 처리되지 않으며**, LOCATION은 `$PGDATA` 하위에 두면 안 됩니다.

---

## 2. CREATE TABLESPACE

```sql
CREATE TABLESPACE testspace
  OWNER chlee
  LOCATION '/home/chlee/workspace/postgresql-12.3/data'
  WITH (tablespace_option = value);
```
- **LOCATION**: 테이블스페이스 파일 생성 위치. **`$PGDATA` 하위 금지**.
- `tablespace_option`: 쿼리 플랜 관련 가중치(seq_page_cost 등).

## 3. ALTER TABLESPACE

```sql
ALTER TABLESPACE testspace RENAME TO testspace2;
ALTER TABLESPACE testspace OWNER TO chlee2;
ALTER TABLESPACE testspace SET seq_page_cost = 1;   -- 플랜 비용 조정
ALTER TABLESPACE testspace RESET seq_page_cost;
```

## 4. DROP TABLESPACE

```sql
DROP TABLESPACE IF EXISTS chlee;
```
> 삭제 조건:
> - owner 또는 superuser만 가능.
> - 속한 **모든 DB 객체가 비어 있어야** 함.
> - 세션의 임시 테이블스페이스로 사용 중이면 삭제 불가.

## 5. 연관 개념

- [[2026-06-14-PG04_(PostgreSQL-데이터베이스-관리)]]
- [[2026-06-14-PG06_(PostgreSQL-스키마-관리-search_path)]]
- [[2026-06-02_(PostgreSQL-객체-용량-조회-쿼리-모음)]]
