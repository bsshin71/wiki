# PostgreSQL "too many connections" 오류

- **카테고리**: #DBMS #PostgreSQL #admin
- **태그**: #PostgreSQL #error #connection #pgbouncer #문제해결
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-pq too many connections for role  XXX - BigDataTeam]]

## 1. 핵심 요약

`too many connections for role XXX`는 role의 `CONNECTION LIMIT` 또는 전체 `max_connections` 초과, **닫히지 않은 연결(커넥션 풀 누수)**, **풀러 설정** 문제로 발생합니다.
임시로는 한도 상향·idle 연결 종료로 해결하고, 근본적으로는 **PgBouncer 등 커넥션 풀러** 도입을 권장합니다.

---

## 2. 원인

1. **연결 제한 초과** — role `CONNECTION LIMIT` 또는 전역 `max_connections` 초과.
2. **닫히지 않은 연결** — 앱이 연결을 닫지 않아 누적(커넥션 풀 미관리).
3. **Pooler 설정** — PgBouncer 등의 max 연결이 너무 낮음.

## 3. 현재 상태 확인

```sql
SELECT rolname, rolconnlimit FROM pg_roles WHERE rolname = 'pmm';     -- role 한도
SELECT pid, usename, client_addr, state FROM pg_stat_activity WHERE usename='pmm';
```

## 4. 한도 상향

```sql
ALTER ROLE pmm CONNECTION LIMIT 50;       -- role별
SHOW max_connections;
ALTER SYSTEM SET max_connections = 500;   -- 전역 (재시작 필요)
SELECT pg_reload_conf();
```

## 5. 불필요한 연결 종료

```sql
SELECT pg_terminate_backend(pid) FROM pg_stat_activity
 WHERE usename='pmm' AND state='idle';
```

## 6. 근본 해결 — PgBouncer (커넥션 풀러)

```ini
[databases]
yourdb = host=127.0.0.1 port=5432 dbname=yourdb pool_size=20
```
> 앱↔PgBouncer↔PostgreSQL 구조로 실제 DB 연결 수를 풀 크기로 제한 → max_connections 압박 완화.

## 7. 연관 개념

- [[2026-06-14-PG08_(PostgreSQL-세션-관리-연결제어)]]
- [[2026-06-14-PG19_(PostgreSQL-모니터링-시스템뷰-pg_stat_activity)]]
