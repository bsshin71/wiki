# PostgreSQL 데이터베이스 관리

- **카테고리**: #DBMS #PostgreSQL #admin
- **태그**: #PostgreSQL #database #template #collation #admin
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-postgresql 데이터베이스 관리 - BigDataTeam]]

## 1. 핵심 요약

PostgreSQL은 multi-database를 지원하며, `CREATE DATABASE`는 **template(template0/template1)을 복사**해 생성합니다.
DB 생성에는 role의 `createdb` 속성이 필요하고, `LC_COLLATE`/`LC_CTYPE`(정렬·문자분류)은 OS 로케일 의존적이라 **한글 정렬 시 주의**가 필요합니다.

---

## 2. CREATE DATABASE

```sql
CREATE DATABASE test2
  OWNER chlee
  ENCODING utf8
  LC_COLLATE 'ko_KR.utf8'      -- 정렬 순서·문자 분류
  LC_CTYPE 'ko_KR.utf8'
  ALLOW_CONNECTIONS true
  CONNECTION_LIMIT 1024
  TABLESPACE testspace
  TEMPLATE template0;
```
> 권한: superuser 또는 `createdb` 속성 role.

### Template
| template | 특징 |
|----------|------|
| template0 | 변경 불가·기본값. **pg_dump 복원 시 유용** |
| template1 | 기본 template. site-local 추가분이 이후 생성 DB의 기본값 |

> `collation`은 OS 의존 → `SELECT * FROM pg_collation;`으로 확인.

## 3. ALTER DATABASE

```sql
ALTER DATABASE chlee ALLOW_CONNECTIONS false;   -- 접근 금지
ALTER DATABASE chlee RENAME TO chlee2;          -- 이름 변경
ALTER DATABASE chlee OWNER TO chlee2;           -- 오너 변경
ALTER DATABASE chlee SET TABLESPACE chleespace; -- 테이블스페이스 변경
ALTER DATABASE chlee SET configuration_parameter TO value;
```

## 4. DROP DATABASE

```sql
DROP DATABASE [IF EXISTS] chlee2;
```
> superuser/owner만 가능, **rollback 불가**, 연결된 세션이 있으면 실패.

## 5. ⚠️ 한글 정렬 이슈

> UTF8 DB의 `LC_COLLATE=utf8` 기준 한글 정렬에 문제가 있을 수 있어, **목적에 맞게 collation 설정** 필요.

## 6. 연관 개념

- [[2026-06-14-PG05_(PostgreSQL-사용자-Role-관리)]]
- [[2026-06-14-PG03_(PostgreSQL-구동-및-종료-pg_ctl)]]
