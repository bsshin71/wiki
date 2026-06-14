# PostgreSQL 세션 관리 (연결 제어)

- **카테고리**: #DBMS #PostgreSQL #admin
- **태그**: #PostgreSQL #session #connection #pg_stat_activity #terminate
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-postgresql 세션 관리 - BigDataTeam]]

## 1. 핵심 요약

세션 현황은 **`pg_stat_activity`** 로 조회하고, 연결 한도는 role별 `CONNECTION LIMIT` / DB 전체 `max_connections`로 제어합니다.
세션 강제 종료는 **`pg_cancel_backend`(쿼리만 취소)** / **`pg_terminate_backend`(세션 종료)** 를 사용합니다.

---

## 2. 연결 수 조회·설정

```sql
-- role별 연결 한도
SELECT rolname, rolconnlimit FROM pg_roles WHERE rolname = 'pmm';
ALTER ROLE pmm CONNECTION LIMIT 50;

-- DB 전체 최대 연결
SHOW max_connections;
ALTER SYSTEM SET max_connections = 500;
SELECT pg_reload_conf();     -- (max_connections는 재시작 필요)
```

## 3. 현재 연결 상태 (pg_stat_activity)

```sql
SELECT pid, usename, client_addr, application_name, state
FROM pg_stat_activity WHERE usename = 'pmm';
-- state: active / idle / idle in transaction ...
```

## 4. 세션/쿼리 종료

```sql
-- 쿼리만 취소(세션 유지)
SELECT pg_cancel_backend(275926);
-- 세션 강제 종료
SELECT pg_terminate_backend(275926);

-- idle 연결 일괄 종료
SELECT pg_terminate_backend(pid) FROM pg_stat_activity
 WHERE usename = 'pmm' AND state = 'idle';

-- 특정 DB의 세션 종료(자기 자신 제외)
SELECT pg_terminate_backend(pid) FROM pg_stat_activity
 WHERE datname = 'useract' AND pid <> pg_backend_pid();
```

## 5. 연관 개념

- [[2026-06-14-PG05_(PostgreSQL-사용자-Role-관리)]]
- [[2026-06-02_(PostgreSQL-모니터링-쿼리-모음)]]
