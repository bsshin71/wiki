# PostgreSQL 상황별 모니터링 쿼리

- **카테고리**: #DBMS #PostgreSQL #admin
- **태그**: #PostgreSQL #모니터링 #용량 #lock_wait #long_query
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-상황별 모니터링 쿼리 - BigDataTeam]]

## 1. 핵심 요약

운영에서 자주 쓰는 모니터링 쿼리 모음으로, **DB/테이블/인덱스 용량**, **실행 중·장기 쿼리**, **lock wait/blocking** 진단 쿼리를 제공합니다.
`pg_blocking_pids()`로 blocking 프로세스를, `pg_locks`로 락 상세를 추적합니다.

---

## 2. 용량 조회

```sql
-- DB별 용량
SELECT datname, pg_size_pretty(pg_database_size(datname)) AS size
FROM pg_database ORDER BY pg_database_size(datname) DESC;

-- 테이블 상위 10 (total/data/external)
SELECT relname, pg_size_pretty(pg_total_relation_size(relid)) AS total_size,
       pg_size_pretty(pg_relation_size(relid)) AS data_size
FROM pg_statio_user_tables ORDER BY pg_total_relation_size(relid) DESC LIMIT 10;

-- 인덱스 크기 / 스캔 횟수
SELECT indexrelname, pg_size_pretty(pg_relation_size(indexrelid)) FROM pg_stat_all_indexes WHERE schemaname='public';
SELECT relname, indexrelname, idx_scan FROM pg_stat_all_indexes WHERE schemaname='public' ORDER BY idx_scan DESC;
```

## 3. 실행 중 / 장기 쿼리

```sql
-- 현재 active 쿼리
SELECT datname, pid, usename, query_start,
       to_char(NOW()-query_start,'HH24:MI:SS') AS runtime, substr(query,1,100)
FROM pg_stat_activity WHERE state='active' ORDER BY query_start;

-- 5분 초과 쿼리
SELECT pid, usename, query_start, now()-query_start AS query_time,
       query, state, wait_event_type, wait_event
FROM pg_stat_activity WHERE (now()-query_start) > interval '5 minutes';
```

## 4. Lock wait / Blocking

```sql
-- blocking 프로세스 찾기 (pg_blocking_pids)
SELECT activity.pid, activity.usename, activity.query,
       blocking.pid AS blocking_id, blocking.query AS blocking_query
FROM pg_stat_activity activity
JOIN pg_stat_activity blocking ON blocking.pid = ANY(pg_blocking_pids(activity.pid));

-- 락 상세 (granted 여부)
SELECT t.relname, l.locktype, l.pid, l.mode, l.granted
FROM pg_locks l JOIN pg_stat_all_tables t ON l.relation = t.relid;
```

## 5. vacuum/analyze 이력

```sql
SELECT relname, last_vacuum, last_autovacuum, last_analyze, last_autoanalyze
FROM pg_stat_user_tables;
```

## 6. 연관 개념

- [[2026-06-14-PG19_(PostgreSQL-모니터링-시스템뷰-pg_stat_activity)]]
- [[2026-06-14-PG21_(PostgreSQL-문제상황-Index-Bloating-Deadlock)]]
- [[2026-06-02_(PostgreSQL-객체-용량-조회-쿼리-모음)]]
