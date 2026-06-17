# MySQL Performance Schema 활용

- **카테고리**: #DBMS #쿼리튜닝
- **태그**: #MySQL #performance_schema #sys #모니터링 #성능분석 #프로파일링
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-MySQL Performance_schema 활용 - BigDataTeam]], [[2026-06-12-Performance 스키마를 이용한 프로파일링 - BigDataTeam]], [[2026-06-12-events_statements_summary_by_digest를 이용하여 SQL성능 분석 - BigDataTeam]], [[2026-06-12-MySQL 성능 튜닝 - BigDataTeam]]

## 1. 핵심 요약

- `performance_schema`는 MySQL 내부 **실시간 성능 모니터링** 도구. 테이블·세션·쿼리 단위 통계 수집.
- `sys` 스키마는 `performance_schema`를 쉽게 사용하는 뷰 모음. 실무에서 가장 먼저 사용.
- `events_statements_summary_by_digest`: 누적 SQL 성능 분석의 핵심. full scan, sort, index 미사용 여부 확인.
- InnoDB 튜닝 핵심: `innodb_buffer_pool_size`(물리 메모리 70~80%), `innodb_redo_log_capacity`(8.0.31+).
- 오버헤드 고려해 운영 환경에서는 **필요한 consumer만 활성화**.

---

## 2. Performance Schema 활성화 및 설정

### 2.1 활성화 확인

```sql
SHOW VARIABLES LIKE 'performance_schema%';
-- performance_schema: ON
```

```ini
# my.cnf (재시작 필요)
[mysqld]
performance_schema=ON
performance_schema_max_table_instances=12500
```

### 2.2 Consumer / Instrument 활성화

```sql
-- 쿼리 프로파일링 활성화 (instrument)
UPDATE performance_schema.setup_instruments
SET enabled='YES', timed='YES'
WHERE name LIKE '%statement/%' OR name LIKE '%stage%';

-- consumer 활성화 (상세 정보 수집)
UPDATE performance_schema.setup_consumers
SET ENABLED='YES'
WHERE NAME IN (
  'events_statements_current',
  'events_statements_history',
  'events_statements_history_long',   -- 기본 비활성
  'events_waits_current',
  'events_waits_history'
);
```

### 2.3 설정 관련 파라미터 (read only, restart 필요)

```text
performance_schema_max_sql_text_length                  : SQL 텍스트 최대 저장 길이 (기본 1024)
performance_schema_max_digest_length                    : DIGEST 텍스트 최대 저장 길이 (기본 1024)
performance_schema_events_statements_history_size       : 스레드당 히스토리 저장 수 (기본 10)
performance_schema_events_statements_history_long_size  : history_long 전체 개수 (기본 10000)
```

---

## 3. 테이블 초기화

```sql
-- 전체 초기화 (sys 편의 함수)
CALL sys.ps_truncate_all_tables(FALSE);
-- 출력: Truncated 49 tables

-- 개별 초기화
TRUNCATE TABLE performance_schema.events_statements_current;
TRUNCATE TABLE performance_schema.events_statements_history;
```

---

## 4. 접속·세션 모니터링

### 4.1 계정·호스트별 접속 통계

```sql
-- 계정별
SELECT USER, HOST, CURRENT_CONNECTIONS, TOTAL_CONNECTIONS
FROM performance_schema.accounts ORDER BY TOTAL_CONNECTIONS DESC;

-- 호스트별 (현재 접속 중인 외부 호스트만)
SELECT HOST, CURRENT_CONNECTIONS, TOTAL_CONNECTIONS
FROM performance_schema.hosts
WHERE current_connections > 0 AND host NOT IN ('NULL','127.0.0.1')
ORDER BY host;

-- User별
SELECT USER, CURRENT_CONNECTIONS, TOTAL_CONNECTIONS
FROM performance_schema.users ORDER BY TOTAL_CONNECTIONS DESC;
```

### 4.2 미사용 DB 계정 확인

```sql
SELECT DISTINCT m_u.user, m_u.host
FROM mysql.user m_u
LEFT JOIN performance_schema.accounts ps_a ON m_u.user=ps_a.user AND ps_a.host=m_u.host
LEFT JOIN information_schema.views is_v ON is_v.definer=concat(m_u.user,'@',m_u.host) AND is_v.security_type='DEFINER'
LEFT JOIN information_schema.routines is_r ON is_r.definer=concat(m_u.user,'@',m_u.host) AND is_r.security_type='DEFINER'
WHERE ps_a.user IS NULL AND is_v.definer IS NULL AND is_r.definer IS NULL
ORDER BY m_u.user, m_u.host;
```

### 4.3 SHOW PROCESSLIST ID ↔ Thread ID 매핑

```sql
SELECT thread_id, processlist_id FROM performance_schema.threads WHERE processlist_id = 1507774;
SELECT sys.ps_thread_id(1507774);
```

### 4.4 현재 활성 세션 (sys.session)

```sql
SELECT thd_id, conn_id, user, db, command, state, time,
       sys.format_time(trx_latency) AS tx_duration, current_statement
FROM sys.session
WHERE thd_id IS NOT NULL
ORDER BY time DESC;

-- 30초 이상 장기 대기 세션만
SELECT thd_id, conn_id, user, db, command, state, time AS wait_sec, current_statement
FROM sys.session
WHERE state NOT IN ('Sleep', NULL) AND time > 30
ORDER BY time DESC;
```

---

## 5. SQL 성능 분석 (events_statements_summary_by_digest)

> ⭐ digest 기반 심화 분석(정렬·임시테이블·에러 감지 변형 쿼리)은 [[2026-06-13-09_(MySQL-SQL성능-분석-Performance-Schema)]] 참조.

### 5.1 지연시간 상위 쿼리 (TOP 5)

```sql
SELECT IF(LENGTH(DIGEST_TEXT) > 64, CONCAT(LEFT(DIGEST_TEXT, 30), ' ... ', RIGHT(DIGEST_TEXT, 30)), DIGEST_TEXT) AS query,
       IF(SUM_NO_GOOD_INDEX_USED > 0 OR SUM_NO_INDEX_USED > 0, '*', '') AS full_scan,
       COUNT_STAR AS exec_count,
       SUM_ERRORS AS err_count, SUM_WARNINGS AS warn_count,
       SEC_TO_TIME(SUM_TIMER_WAIT/1000000000000) AS exec_time_total,
       SEC_TO_TIME(MAX_TIMER_WAIT/1000000000000) AS exec_time_max,
       (AVG_TIMER_WAIT/1000000000) AS exec_time_avg_ms,
       SUM_ROWS_SENT AS rows_sent,
       ROUND(SUM_ROWS_SENT/COUNT_STAR) AS rows_sent_avg,
       SUM_ROWS_EXAMINED AS rows_scanned
FROM performance_schema.events_statements_summary_by_digest
WHERE SCHEMA_NAME='DB명'
ORDER BY SUM_TIMER_WAIT DESC LIMIT 5;
```

### 5.2 임시 테이블 (정렬 최적화 확인)

```sql
-- disk_tmp_tables 사용률 높으면 sort_buffer_size / tmp_table_size 증설 고려
SELECT IF(LENGTH(DIGEST_TEXT) > 64, CONCAT(LEFT(DIGEST_TEXT, 30), ' ... ', RIGHT(DIGEST_TEXT, 30)), DIGEST_TEXT) AS query,
       COUNT_STAR AS exec_count,
       SUM_CREATED_TMP_TABLES AS memory_tmp_tables,
       SUM_CREATED_TMP_DISK_TABLES AS disk_tmp_tables,
       ROUND(SUM_CREATED_TMP_DISK_TABLES/SUM_CREATED_TMP_TABLES*100) AS tmp_tables_to_disk_pct
FROM performance_schema.events_statements_summary_by_digest
WHERE SUM_CREATED_TMP_TABLES > 0 AND SCHEMA_NAME='DB명'
ORDER BY SUM_CREATED_TMP_DISK_TABLES DESC LIMIT 5;
```

### 5.3 인덱스 미사용 쿼리 (Full Scan)

```sql
SELECT IF(LENGTH(DIGEST_TEXT) > 64, CONCAT(LEFT(DIGEST_TEXT, 30), ' ... ', RIGHT(DIGEST_TEXT, 30)), DIGEST_TEXT) AS query,
       COUNT_STAR AS exec_count,
       SUM_NO_INDEX_USED AS no_index_used_count,
       ROUND((SUM_NO_INDEX_USED/COUNT_STAR)*100) AS no_index_used_pct
FROM performance_schema.events_statements_summary_by_digest
WHERE (SUM_NO_INDEX_USED > 0 OR SUM_NO_GOOD_INDEX_USED > 0) AND SCHEMA_NAME='DB명'
ORDER BY no_index_used_pct DESC, exec_count DESC LIMIT 5;
```

---

## 6. sys 스키마를 활용한 성능 분석

```sql
-- 실행시간 긴 쿼리 TOP 10
SELECT query, exec_count,
       sys.format_time(avg_latency) AS formatted_avg_latency,
       rows_sent_avg, rows_examined_avg, last_seen
FROM sys.x$statement_analysis
ORDER BY avg_latency DESC LIMIT 10;

-- Full Table Scan 쿼리
SELECT db, query, exec_count,
       sys.format_time(total_latency) AS formatted_total_latency,
       rows_examined_avg, last_seen
FROM sys.x$statements_with_full_table_scans
WHERE db='DB명'
ORDER BY total_latency DESC LIMIT 10;

-- 자주 실행되는 쿼리
SELECT db, exec_count, query FROM sys.statement_analysis ORDER BY exec_count DESC LIMIT 10;

-- 95th percentile 느린 쿼리
SELECT * FROM sys.statements_with_runtimes_in_95th_percentile LIMIT 10;

-- 파일 I/O 상위
SELECT * FROM sys.io_global_by_file_by_latency LIMIT 10;
SELECT * FROM sys.io_global_by_file_by_bytes WHERE file LIKE '%ibd%';
```

### 6.1 주요 sys 뷰 정리

| 뷰 이름 | 용도 |
|---------|------|
| `sys.statements_with_full_table_scans` | Full Scan 쿼리 |
| `sys.io_global_by_file_by_latency` | 파일 I/O 상위 |
| `sys.io_global_by_wait_by_latency` | 대기 이벤트 상위 |
| `sys.host_summary` / `sys.user_summary` | 호스트·계정별 통계 |
| `sys.innodb_lock_waits` | InnoDB Lock 현황 |
| `sys.schema_unused_indexes` | 미사용 인덱스 |

### 6.2 sys 포맷 함수

```sql
SELECT sys.format_time(12345678910) AS formatted_time,   -- 12.35 s
       sys.format_bytes(1073741824) AS formatted_bytes;  -- 1.00 GiB
```

---

## 7. 인덱스 효율성 분석

> ⭐ 인덱스 설계·최적화 상세는 [[2026-06-13-25_(MySQL-Index-최적화)]], 대용량 인덱스 생성은 [[2026-06-13-10_(MySQL-Fast-Index-Creation-최적화)]] 참조.

```sql
-- 미사용 인덱스 식별
SELECT object_schema, object_name, index_name,
       count_read, count_write, count_delete, count_update, count_insert
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE count_read = 0 AND count_write = 0 AND index_name != 'PRIMARY'
ORDER BY object_schema, object_name;

-- sys 편의 뷰 (미사용/중복 인덱스)
SELECT * FROM sys.schema_unused_indexes WHERE object_schema='DB명';
SELECT * FROM sys.schema_redundant_indexes \G;

-- 미사용 인덱스 안전 테스트 (Invisible 전환)
ALTER TABLE 테이블명 ALTER INDEX 인덱스명 INVISIBLE;

-- 인덱스별 I/O 통계
SELECT object_schema, object_name, index_name, count_read, count_write,
       SUM_TIMER_READ/1e12 AS read_time_sec, SUM_TIMER_WRITE/1e12 AS write_time_sec
FROM performance_schema.table_io_waits_summary_by_index_usage
ORDER BY SUM_TIMER_WAIT DESC LIMIT 20;

-- 인덱스 통계 (선택/삽입/수정/삭제 횟수)
SELECT * FROM sys.schema_index_statistics LIMIT 10 \G;
```

---

## 8. 테이블 I/O 분석

```sql
-- 전체 테이블 I/O 통계
SELECT object_schema, object_name,
       count_read, count_write, count_delete, count_insert, count_update,
       sum_timer_wait/1e12 AS total_wait_sec
FROM performance_schema.table_io_waits_summary_by_table
ORDER BY sum_timer_wait DESC LIMIT 20;

-- 테이블별 row 변경 추적 (sys)
SELECT table_schema, table_name,
       rows_fetched, rows_inserted, rows_updated, rows_deleted, io_read, io_write
FROM sys.schema_table_statistics
WHERE table_schema NOT IN ('mysql','performance_schema','sys');

-- Full scan 발생 테이블
SELECT * FROM sys.schema_tables_with_full_table_scans;

-- auto_increment 사용률
SELECT table_schema, table_name, column_name,
       auto_increment AS current_value, max_value,
       ROUND(auto_increment_ratio*100, 2) AS usage_ratio
FROM sys.schema_auto_increment_columns;
```

---

## 9. 쿼리 프로파일링 (단계별 소요시간 추적)

```sql
-- 1. 현재 설정 저장
CALL sys.ps_setup_save(10);

-- 2. 프로파일링 활성화
UPDATE performance_schema.setup_instruments SET enabled='YES', timed='YES'
WHERE name LIKE '%statement/%' OR name LIKE '%stage%';
UPDATE performance_schema.setup_consumers SET enabled='YES'
WHERE name LIKE '%events_statements_%' OR name LIKE '%events_stages_%';

-- 3. 대상 쿼리 실행
SELECT * FROM tb_test1;

-- 4. EVENT_ID 확인
SELECT event_id, sql_text, sys.format_time(TIMER_WAIT) AS duration
FROM performance_schema.events_statements_history_long
WHERE SQL_TEXT LIKE '%tb_test1%';
-- 예: event_id=114, duration=185.84 us

-- 5. 단계별(stage) 상세 확인
SELECT event_name AS stage, sys.format_time(TIMER_WAIT) AS duration
FROM performance_schema.events_stages_history_long
WHERE nesting_event_id=114   -- 위에서 확인한 EVENT_ID
ORDER BY timer_start;
-- stage/sql/starting → checking permissions → Opening tables → optimizing → executing ...

-- 6. 설정 복구
CALL sys.ps_setup_reload_saved();
```

---

## 10. 메모리 사용 분석

```sql
-- MySQL 총 메모리 사용량
SELECT * FROM sys.memory_global_total;   -- 예: 16.46 GiB

-- 스레드별 메모리 사용량
SELECT thread_id, current_allocated FROM sys.memory_by_thread_by_current_bytes;

-- 모듈(code area)별 메모리 사용량
SELECT SUBSTRING_INDEX(event_name,'/',2) AS code_area,
       sys.format_bytes(SUM(current_alloc)) AS current_alloc
FROM sys.x$memory_global_by_current_bytes
GROUP BY SUBSTRING_INDEX(event_name,'/',2)
ORDER BY SUM(current_alloc) DESC;
-- memory/innodb 가 보통 가장 큼

-- 이벤트별 글로벌 메모리
SELECT event_name,
       current_alloc/1024/1024/1024 AS allocated_gb,
       number_of_bytes_alloc, number_of_bytes_free
FROM sys.x$memory_global_by_current_bytes
ORDER BY current_alloc DESC LIMIT 20;

-- 특정 스레드 메모리 상세
SELECT thread_id, event_name,
       sys.format_bytes(CURRENT_NUMBER_OF_BYTES_USED) AS current_allocated
FROM performance_schema.memory_summary_by_thread_by_event_name
WHERE thread_id=46
ORDER BY CURRENT_NUMBER_OF_BYTES_USED DESC LIMIT 10;
```

---

## 11. InnoDB 성능 튜닝 핵심

```sql
-- Redo Log 용량 조정 (8.0.31+, 동적)
SET GLOBAL innodb_redo_log_capacity = 12 * 1024 * 1024 * 1024;  -- 12GB

-- Buffer Pool 크기: 물리 메모리의 70~80%
-- buffer_pool_instances: CPU 코어 수/4 (최대 8). 증가 효과는 throughput보다
--   max latency 이상치 빈도 감소.

-- 현재 InnoDB 파라미터 확인
SHOW VARIABLES LIKE 'innodb_buffer_pool%';
SHOW VARIABLES LIKE 'innodb_flush%';
SHOW VARIABLES LIKE 'innodb_log%';
SHOW ENGINE INNODB STATUS \G;

-- InnoDB Buffer Pool 히트율 (테이블별)
SELECT object_schema, object_name, count_read, count_read_dup,
       IF(count_read=0, 0, count_read_dup/count_read*100) AS hit_rate_pct
FROM performance_schema.table_io_waits_summary_by_table
WHERE object_schema NOT IN ('mysql','information_schema')
ORDER BY hit_rate_pct DESC;
```

---

## 12. Performance Schema 성능 영향 (오버헤드)

| 설정 | 오버헤드 |
|------|----------|
| OFF | 없음 ✅ |
| ON (setup_consumers OFF) | ~3-5% |
| ON (statements 수집) | ~5-10% |
| ON (모든 이벤트 수집) | 10-15% |

**권장**: 운영 환경에서는 **필요한 consumer만 활성화**. 분석 후에는 다시 OFF 또는 최소화.

**리셋 후 재분석 절차:**
```sql
CALL sys.ps_truncate_all_tables(FALSE);   -- 1. 통계 초기화
-- 2. 충분한 시간(예: 1시간) 운영
SELECT * FROM sys.statements_with_full_table_scans LIMIT 10;  -- 3. 분석
```

---

## 13. 연관 개념

- [[2026-06-13-09_(MySQL-SQL성능-분석-Performance-Schema)]] — digest 기반 SQL 성능 심화 분석
- [[2026-06-13-25_(MySQL-Index-최적화)]] — 인덱스 설계·최적화
- [[2026-06-13-10_(MySQL-Fast-Index-Creation-최적화)]] — 대용량 인덱스 생성 최적화
- [[2026-06-13_(MySQL-Lock-Deadlock-모니터링)]] — Lock 모니터링 (performance_schema 활용)
- [[2026-06-13_(MySQL-관리자-쿼리-모음)]] — 관리자 운영 쿼리 모음
