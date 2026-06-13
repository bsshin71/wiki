# MySQL Meta Lock 세션 조회

- **카테고리**: #DBMS #MySQL #Lock
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-meta lock 을 잡고 있는 세션찾기 - BigDataTeam.md]]

## 1. 핵심 요약

Meta Lock(MDL)을 잡고 있는 세션을 찾기 위해 performance_schema.threads와 data_locks를 조인합니다.
ALTER TABLE, DROP TABLE 등 DDL이 블로킹될 때 MDL_EXCLUSIVE를 점유한 세션의 PID를 찾아 강제 종료합니다.

---

## 2. Meta Lock 세션 조회

```sql
SELECT
  t.PROCESSLIST_ID AS pid,
  t.PROCESSLIST_USER AS user,
  t.PROCESSLIST_HOST AS host,
  t.PROCESSLIST_DB AS db,
  t.PROCESSLIST_TIME AS elapsed_sec,
  t.PROCESSLIST_STATE AS state,
  es.SQL_TEXT AS current_sql,
  dl.LOCK_MODE,
  dl.OBJECT_SCHEMA,
  dl.OBJECT_NAME
FROM performance_schema.threads t
JOIN performance_schema.data_locks dl ON t.thread_id = dl.thread_id
LEFT JOIN performance_schema.events_statements_current es ON es.thread_id = dl.thread_id
WHERE dl.LOCK_MODE IN ('MDL_EXCLUSIVE', 'MDL_SHARED')
ORDER BY t.PROCESSLIST_TIME DESC;
```

---

## 3. 특정 테이블의 Meta Lock

```sql
SELECT t.PROCESSLIST_ID, t.PROCESSLIST_USER, t.PROCESSLIST_TIME,
       es.SQL_TEXT, dl.LOCK_MODE
FROM performance_schema.threads t
JOIN performance_schema.data_locks dl ON t.thread_id = dl.thread_id
LEFT JOIN performance_schema.events_statements_current es ON es.thread_id = dl.thread_id
WHERE dl.OBJECT_SCHEMA = 'mydb' 
  AND dl.OBJECT_NAME = 'mytable'
  AND dl.LOCK_MODE LIKE 'MDL%';
```

---

## 4. 세션 강제 종료

```sql
KILL QUERY 12345;    -- 현재 쿼리만 취소
KILL SESSION 12345;  -- 전체 세션 종료
```

---

## 5. 연관 개념

- [[2026-06-13-16_(MySQL-Lock-개념-및-분석)]]
