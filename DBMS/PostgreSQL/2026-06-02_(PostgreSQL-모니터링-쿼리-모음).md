# PostgreSQL 모니터링 쿼리 모음
- **카테고리**: #DBMS #PostgreSQL
- **태그**: #PostgreSQL #admin #모니터링 #pg_stat_statements #쿼리모음
- **작성일**: 2026-06-02
- **참조 원본**: [[2026-06-02-pg_stat_statements 활용]], [[2026-06-02-실행중인 쿼리 확인]], [[2026-06-02-database 별 cache hit  miss 비율 조회]], [[2026-06-02-PostgreSQL 통계 정보관련 view 테이블 종류]]

## 1. 핵심 요약
- PostgreSQL 운영 중 성능 모니터링에 사용하는 핵심 쿼리 모음. pg_stat_statements, pg_stat_activity, 세션·Lock·Cache Hit 조회 포함.

## 2. 상세 설명

### 주요 통계 뷰 종류

| 뷰 | 설명 |
|----|------|
| `pg_stat_activity` | 현재 실행 중인 모든 세션 정보 (쿼리, 상태, 대기 이벤트) |
| `pg_stat_user_tables` | 사용자 테이블별 통계 (Dead Tuple, Vacuum 시간 등) |
| `pg_stat_user_indexes` | 사용자 인덱스별 통계 |
| `pg_stat_database` | 데이터베이스별 통계 |
| `pg_stat_statements` | 실행된 SQL의 성능 통계 (확장 설치 필요) |
| `pg_stat_bgwriter` | Background Writer 통계 |

---

### 실행 중인 쿼리 확인

```sql
-- 현재 active 세션의 쿼리 확인
SELECT pid, usename, query, state, query_start
FROM pg_stat_activity
WHERE state = 'active';

-- Blocking 관계 조회
SELECT
    blocked.pid,
    blocked.usename,
    blocked.query       AS blocked_query,
    blocking.pid        AS blocking_pid,
    blocking.query      AS blocking_query
FROM pg_stat_activity AS blocked
JOIN pg_stat_activity AS blocking
    ON blocking.pid = ANY(pg_blocking_pids(blocked.pid))
WHERE CARDINALITY(pg_blocking_pids(blocked.pid)) > 0;
```

---

### pg_stat_statements 활용 (느린 SQL 탐지)

```sql
-- 총 수행시간 상위 10개 쿼리 (초 단위)
SELECT
    query,
    calls                                     AS "호출횟수",
    total_exec_time / 1000                    AS "총 소요시간(초)",
    rows,
    100.0 * total_exec_time / sum(total_exec_time) OVER () AS "총 실행시간의 백분율",
    mean_exec_time / 1000                     AS "평균 소요시간(초)",
    min_exec_time / 1000                      AS "최소 소요시간(초)",
    max_exec_time / 1000                      AS "최대 소요시간(초)"
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- 유저명, DB명 포함
SELECT
    rolname AS username,
    datname AS database_name,
    query,
    calls,
    total_exec_time / 1000  AS "총 소요시간(초)",
    max_exec_time / 1000    AS "최대 소요시간(초)"
FROM pg_stat_statements
JOIN pg_database ON pg_stat_statements.dbid = pg_database.oid
JOIN pg_roles    ON pg_stat_statements.userid = pg_roles.oid
ORDER BY total_exec_time DESC
LIMIT 10;

-- 최대 수행시간 상위 50개
SELECT a.userid, b.usename, a.dbid, c.datname
     , a.queryid, substr(a.query, 1, 100) AS query
     , a.calls, a.total_exec_time, a.max_exec_time, a.rows
FROM public.pg_stat_statements a
JOIN pg_catalog.pg_user b         ON a.userid = b.usesysid
JOIN pg_catalog.pg_stat_database c ON a.dbid  = c.datid
ORDER BY a.max_exec_time DESC
LIMIT 50;
```

---

### Database Cache Hit/Miss 비율

```sql
SELECT datname, blks_hit, blks_read
FROM pg_stat_database;
```

> `blks_hit / (blks_hit + blks_read)` 비율이 높을수록 캐시 효율 좋음.

---

### pg_sleep을 이용한 세션 테스트 패턴

```sql
-- 터미널 1: 60초 대기 세션 생성
SELECT pg_sleep(60);

-- 터미널 2: 실행 중인 세션 확인
SELECT pid, usename, query, state, query_start
FROM pg_stat_activity
WHERE state = 'active';
```

## 3. 연관 개념 (지식 연결)
- [[2026-06-08_(Advanced-SQL-PostgreSQL)]] — PostgreSQL EXPLAIN·성능 분석 상세
- [[2026-06-02_(PostgreSQL-EXPLAIN-실행계획-가이드)]] — EXPLAIN 옵션 상세
