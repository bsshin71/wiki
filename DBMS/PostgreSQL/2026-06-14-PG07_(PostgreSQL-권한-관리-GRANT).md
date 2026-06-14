# PostgreSQL 권한 관리 (GRANT)

- **카테고리**: #DBMS #PostgreSQL #admin
- **태그**: #PostgreSQL #권한 #GRANT #role #membership
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-postgresql 권한 관리 - BigDataTeam]]

## 1. 핵심 요약

객체 권한은 owner/superuser가 `GRANT`로 다른 role에 부여하며, 객체 종류별로 부여 가능한 권한이 다릅니다(예 TABLE=arwdDxt, DATABASE=CTc).
Role 멤버십은 `GRANT role TO role`로 부여하고, `WITH GRANT/ADMIN OPTION`으로 재위임 권한을, `SET ROLE`로 그룹 권한을 활성화합니다(단 `NOINHERIT` role은 SET ROLE 필요).

---

## 2. Privilege 종류

`SELECT, INSERT, UPDATE, DELETE, TRUNCATE, REFERENCES, TRIGGER, CREATE, CONNECT, TEMPORARY, EXECUTE, USAGE` + `ALL PRIVILEGES`.

## 3. 객체별 부여 가능 권한 (요약)

| Object | All Privileges | Default PUBLIC |
|--------|---------------|----------------|
| DATABASE | CTc | Tc |
| SCHEMA | UC | none |
| TABLE | arwdDxt | none |
| SEQUENCE | rwU | none |
| FUNCTION/PROCEDURE | X | X |
| TABLESPACE | C | none |

> 약어: r=SELECT, a=INSERT, w=UPDATE, d=DELETE, D=TRUNCATE, x=REFERENCES, t=TRIGGER, C=CREATE, c=CONNECT, T=TEMPORARY, X=EXECUTE, U=USAGE.

## 4. 객체 권한 부여

```sql
GRANT INSERT ON films TO PUBLIC;           -- 모든 유저
GRANT ALL PRIVILEGES ON kinds TO manuel;
GRANT ALL PRIVILEGES ON DATABASE "my_db" TO my_user;
```
- **WITH GRANT OPTION**: 부여받은 권한을 다른 role에 재부여 가능.

## 5. Role 멤버십

```sql
GRANT admins TO joe;     -- joe에게 admins 그룹 멤버십(권한) 부여
```
- **WITH ADMIN OPTION**: 멤버십을 다른 role에 재부여 가능.
- **GRANTED BY**: 권한 수행 시 명시 role로 기록(superuser 전용).

### SET ROLE (그룹 권한 활성화)
```sql
SET ROLE admin;    -- 직접 멤버십(INHERIT) → 권한 즉시 사용
SET ROLE wheel;    -- NOINHERIT 그룹은 SET ROLE로 전환해야 사용
SET ROLE NONE;     -- 또는 RESET ROLE / SET ROLE joe (되돌리기)
```
> ⚠️ SET ROLE로 **접속 시 지정한 데이터베이스는 바꿀 수 없음** → cross-database 참조는 오류.

## 6. 연관 개념

- [[2026-06-14-PG05_(PostgreSQL-사용자-Role-관리)]]
- [[2026-06-14-PG06_(PostgreSQL-스키마-관리-search_path)]]
- [[2026-06-14-PG04_(PostgreSQL-데이터베이스-관리)]]
