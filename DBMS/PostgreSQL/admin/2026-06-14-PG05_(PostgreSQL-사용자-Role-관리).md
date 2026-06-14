# PostgreSQL 사용자(Role) 관리

- **카테고리**: #DBMS #PostgreSQL #admin
- **태그**: #PostgreSQL #role #user #권한 #REASSIGN
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-postgresql 사용자 관리 - BigDataTeam]]

## 1. 핵심 요약

PostgreSQL은 **user와 group을 Role로 통합**해 관리합니다(`CREATE ROLE` = login 권한 없는 user, login 권한 부여 시 user와 동일).
Role은 SUPERUSER·CREATE ROLE·CREATE DB·REPLICATION 등 속성으로 권한을 정의하며, role 삭제 시 소유 객체는 **REASSIGN/DROP OWNED** 절차로 처리합니다.

---

## 2. USER/Role 조회

```sql
SELECT * FROM PG_SHADOW;   -- 또는 psql에서 \du
```
> 기본 superuser는 `postgres`(SUPERUSER, CREATE ROLE, CREATE DB, REPLICATION 보유).

| 속성 | 기능 |
|------|------|
| SUPERUSER | user 생성·권한 부여 |
| CREATE ROLE | 새 role 생성 |
| CREATE DB | DB 생성 |
| REPLICATION | 실시간 복제 |

## 3. CREATE / ALTER / DROP ROLE

```sql
CREATE ROLE name SUPERUSER;
CREATE ROLE name SUPERUSER, CREATEDB;

ALTER ROLE chlee WITH SUPERUSER CREATEDB INHERIT;  -- 속성 추가
ALTER ROLE chlee WITH NOCREATEDB;                  -- 속성 제거
ALTER ROLE ALL IN DATABASE test SET tcp_keepalives_idle TO DEFAULT;  -- 세션 파라미터

DROP ROLE IF EXISTS chlee2, chlee3;
```

## 4. ⚠️ Role 삭제 시 객체 이양 (REASSIGN / DROP OWNED)

> role은 삭제하되 소유 객체는 유지하려면 — A=삭제 role, B=이어받을 role.
> **각 쿼리는 데이터베이스 단위로 동작**(클러스터 내 모든 DB에서 반복).

```sql
REASSIGN OWNED BY A TO B;   -- A의 객체 소유권을 B로 이양
DROP OWNED BY A;            -- A가 소유한(권한 등) 잔여 정리
-- 클러스터 내 각 데이터베이스에서 위 2줄 반복
DROP ROLE A;
```

## 5. 연관 개념

- [[2026-06-14-PG04_(PostgreSQL-데이터베이스-관리)]]
- [[2026-06-14-PG02_(PostgreSQL-아키텍처-및-특징)]]
