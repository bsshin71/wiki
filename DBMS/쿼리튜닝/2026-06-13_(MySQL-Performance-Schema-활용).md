# MySQL Performance Schema 활용

- **카테고리**: #DBMS #쿼리튜닝 #MySQL #PerformanceSchema
- **작성일**: 2026-06-13
- **참조 원본**: 8개 Performance Schema 관련 소스 파일

## 1. 핵심 요약

Performance Schema는 MySQL의 내부 **실시간 성능 모니터링** 도구로, 테이블 단위·세션 단위·쿼리 단위의 상세 통계를 수집합니다.
SYS 스키마와 함께 사용하면 SQL 성능 분석, 인덱스 효율성 판단, 메모리 사용 분석이 가능합니다.
Events, Statements, Tables, Connections 등 다양한 관점에서 병목을 식별할 수 있습니다.

---

## 2. Performance Schema 활성화

### 2.1 확인

```sql
SHOW VARIABLES LIKE 'performance_schema%';
-- performance_schema: ON

SHOW VARIABLES LIKE 'setup_instruments';
-- 계측 대상 확인
```

### 2.2 활성화 (필요 시)

```ini
[mysqld]
performance_schema=ON
performance_schema_max_table_instances=12500
```

### 2.3 Consumer 활성화

SQL_TEXT 등 상세 정보를 수집하려면:

```sql
UPDATE performance_schema.setup_consumers
SET ENABLED='YES'
WHERE NAME IN (
  'events_statements_current',
  'events_statements_history',
  'events_statements_history_long',
  'events_waits_current',
  'events_waits_history'
);
```

---

## 3. Performance Schema 테이블 초기화

### 3.1 전체 초기화

```sql
-- sys 스키마의 편의 함수 사용
CALL sys.ps_truncate_all_tables(FALSE);

-- 출력: Truncated 49 tables
```

### 3.2 개별 테이블 초기화

```sql
TRUNCATE TABLE performance_schema.events_statements_current;
TRUNCATE TABLE performance_schema.events_statements_history;
```

---

## 4. 접속 정보 분석

### 4.1 계정·호스트별 접속 통계

```sql
-- 계정별
SELECT
  USER,
  HOST,
  CURRENT_CONNECTIONS,
  TOTAL_CONNECTIONS
FROM performance_schema.accounts
ORDER BY TOTAL_CONNECTIONS DESC;

-- 호스트별
SELECT
  HOST,
  CURRENT_CONNECTIONS,
  TOTAL_CONNECTIONS
FROM performance_schema.hosts
ORDER BY TOTAL_CONNECTIONS DESC;
```

### 4.2 User별 접속 통계

```sql
SELECT
  USER,
  CURRENT_CONNECTIONS,
  TOTAL_CONNECTIONS
FROM performance_schema.users
ORDER BY TOTAL_CONNECTIONS DESC;
```

---

## 5. SQL 성능 분석

### 5.1 events_statements_summary_by_digest 활용

```sql
-- 정규화된 쿼리별 성능 통계
SELECT
  DIGEST_TEXT,                    -- 정규화된 SQL
  COUNT_STAR AS exec_count,       -- 실행 횟수
  AVG_TIMER_WAIT/1e12 AS avg_latency_sec,  -- 평균 응답시간
  SUM_ROWS_EXAMINED AS total_rows_examined,
  SUM_ROWS_SENT AS total_rows_sent,
  FIRST_SEEN,
  LAST_SEEN
FROM performance_schema.events_statements_summary_by_digest
WHERE DIGEST_TEXT IS NOT NULL
ORDER BY SUM_TIMER_WAIT DESC    -- 누적 시간 상위
LIMIT 10;
```

### 5.2 sys 스키마의 편의 뷰

```sql
-- Top 10 queries by latency
SELECT *
FROM sys.statements_with_full_table_scans
LIMIT 10;

-- Slow queries
SELECT *
FROM sys.statements_with_runtimes_in_95th_percentile
LIMIT 10;

-- File I/O 상위
SELECT *
FROM sys.io_global_by_file_by_latency
LIMIT 10;
```

---

## 6. 인덱스 효율성 분석

### 6.1 사용되지 않는 인덱스 식별

```sql
SELECT
  object_schema,
  object_name,
  index_name,
  count_read,
  count_write,
  count_delete,
  count_update,
  count_insert
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE count_read = 0
  AND count_write = 0
  AND index_name != 'PRIMARY'
ORDER BY object_schema, object_name;
```

### 6.2 인덱스별 I/O 통계

```sql
SELECT
  object_schema,
  object_name,
  index_name,
  count_read,
  count_write,
  SUM_TIMER_READ/1e12 AS read_time_sec,
  SUM_TIMER_WRITE/1e12 AS write_time_sec
FROM performance_schema.table_io_waits_summary_by_index_usage
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 20;
```

---

## 7. 테이블 I/O 분석

### 7.1 전체 테이블 I/O 통계

```sql
SELECT
  object_schema,
  object_name,
  count_read,
  count_write,
  count_delete,
  count_insert,
  count_update,
  sum_timer_wait/1e12 AS total_wait_sec
FROM performance_schema.table_io_waits_summary_by_table
ORDER BY sum_timer_wait DESC
LIMIT 20;
```

### 7.2 테이블별 row 변경 추적

```sql
SELECT
  object_schema,
  object_name,
  count_insert,
  count_update,
  count_delete,
  count_insert + count_update + count_delete AS total_writes
FROM performance_schema.table_io_waits_summary_by_table
WHERE count_insert + count_update + count_delete > 0
ORDER BY total_writes DESC
LIMIT 20;
```

---

## 8. InnoDB 성능 튜닝

### 8.1 InnoDB Buffer Pool 히트율

```sql
SELECT
  object_schema,
  object_name,
  count_read,
  count_read_dup,
  IF(count_read = 0, 0, count_read_dup / count_read * 100) AS hit_rate_pct
FROM performance_schema.table_io_waits_summary_by_table
WHERE object_schema NOT IN ('mysql', 'information_schema')
ORDER BY hit_rate_pct DESC;
```

### 8.2 InnoDB 페이지 I/O 분석

```sql
-- innodb_buffer_waits 확인 (MySQL 5.7+)
SELECT
  object_name,
  count_read,
  count_write,
  sum_timer_read/1e12 AS read_time_sec,
  sum_timer_write/1e12 AS write_time_sec
FROM performance_schema.table_io_waits_summary_by_table
WHERE engine = 'InnoDB'
ORDER BY sum_timer_wait DESC;
```

---

## 9. 메모리 사용 분석

### 9.1 계정별 메모리 사용

```sql
SELECT
  user,
  current_allocated,
  current_allocated / 1024 / 1024 / 1024 AS allocated_gb,
  current_number_of_bytes_used / 1024 / 1024 / 1024 AS used_gb
FROM performance_schema.memory_summary_by_user_by_event_name
WHERE user IS NOT NULL
GROUP BY user
ORDER BY current_allocated DESC;
```

### 9.2 이벤트별 메모리 사용

```sql
SELECT
  event_name,
  current_allocated / 1024 / 1024 / 1024 AS allocated_gb,
  current_number_of_bytes_used / 1024 / 1024 / 1024 AS used_gb,
  number_of_bytes_alloc,
  number_of_bytes_free
FROM performance_schema.memory_summary_global_by_event_name
WHERE current_allocated > 0
ORDER BY current_allocated DESC
LIMIT 20;
```

---

## 10. 접속·세션 모니터링

### 10.1 현재 활성 세션 (SYS 스키마)

```sql
SELECT
  thd_id,
  conn_id,
  user,
  db,
  command,
  state,
  time,
  sys.format_time(trx_latency) AS tx_duration,
  current_statement
FROM sys.session
WHERE thd_id IS NOT NULL
ORDER BY time DESC;
```

### 10.2 장기 대기 중인 세션

```sql
SELECT
  thd_id,
  conn_id,
  user,
  db,
  command,
  state,
  time AS wait_sec,
  current_statement
FROM sys.session
WHERE state NOT IN ('Sleep', NULL)
  AND time > 30  -- 30초 이상 대기
ORDER BY time DESC;
```

---

## 11. sys 스키마 활용

### 11.1 주요 sys 뷰

| 뷰 이름 | 용도 |
|---------|------|
| `sys.statements_with_full_table_scans` | Full Scan 쿼리 |
| `sys.io_global_by_file_by_latency` | 파일 I/O 상위 |
| `sys.io_global_by_wait_by_latency` | 대기 이벤트 상위 |
| `sys.host_summary` | 호스트별 통계 |
| `sys.user_summary` | 계정별 통계 |
| `sys.innodb_lock_waits` | InnoDB Lock 현황 |
| `sys.schema_unused_indexes` | 미사용 인덱스 |

### 11.2 sys 함수

```sql
-- 포맷팅 함수
SELECT
  sys.format_time(12345678910) AS formatted_time,     -- 시간 형식
  sys.format_bytes(1073741824) AS formatted_bytes;    -- 용량 형식

-- 출력: 12.35 s, 1.00 GiB
```

---

## 12. Performance Schema 성능 영향

| 설정 | 오버헤드 |
|------|----------|
| OFF | 없음 ✅ |
| ON (setup_consumers OFF) | ~3-5% |
| ON (statements 수집) | ~5-10% |
| ON (모든 이벤트 수집) | 10-15% |

**권장**: 운영 환경에서는 **필요한 consumers만 활성화**

---

## 13. Performance Schema 리셋 후 다시 분석

```sql
-- 1. 통계 초기화
CALL sys.ps_truncate_all_tables(FALSE);

-- 2. 충분한 시간 대기 (예: 1시간 운영)

-- 3. 분석 실행
SELECT * FROM sys.statements_with_full_table_scans LIMIT 10;
```

---

## 14. 연관 개념

- [[2026-06-13_(MySQL-Lock-Deadlock-모니터링)]]
- [[2026-06-13_(MySQL-InnoDB-구조-설정)]]
- [[2026-06-13_(MySQL-관리자-쿼리-모음)]]
