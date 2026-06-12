# MySQL Performance Schema 활용
- **카테고리**: #DBMS #쿼리튜닝
- **태그**: #MySQL #performance_schema #모니터링 #성능분석 #sys #프로파일링 #Index #Bulk-Load
- **작성일**: 2026-06-12
- **참조 원본**: [[2026-06-12-MySQL Performance_schema 활용 - BigDataTeam]], [[2026-06-12-Performance 스키마를 이용한 프로파일링 - BigDataTeam]], [[2026-06-12-events_statements_summary_by_digest를 이용하여 SQL성능 분석 - BigDataTeam]], [[2026-06-12-MySQL 성능 튜닝 - BigDataTeam]], [[2026-06-12-mysql optimization Index - BigDataTeam]], [[2026-06-12-fast index creation - BigDataTeam]], [[2026-06-12-성능 최적화를 위한 주요 커널파라미터 - BigDataTeam]], [[2026-06-12-데이터 Load 성능 올리기 - BigDataTeam]], [[2026-06-12-단순 Update문 지연사례 분석 - BigDataTeam]], [[2026-06-12-Update 문 지연2 - BigDataTeam]], [[2026-06-12-03. MySQL admin 쿼리 - BigDataTeam]], [[2026-06-12-mymon shell script for monitoring MySQL.md]]

## 1. 핵심 요약
- `sys` 스키마는 `performance_schema`를 쉽게 사용하는 뷰 모음. 실무에서 가장 먼저 사용.
- `events_statements_summary_by_digest`: 누적 SQL 성능 분석의 핵심. full scan, sort, index 사용 여부 확인 가능.
- InnoDB 성능 튜닝 핵심: `innodb_buffer_pool_size` (물리 메모리 70~80%), `innodb_redo_log_capacity` (8.0.31+).
- Bulk Load 시: binlog OFF + autocommit OFF + constraint OFF + LOAD DATA 사용으로 20x 이상 속도 향상.

## 2. 상세 설명

### Performance Schema 초기화 및 설정

```sql
-- 전체 테이블 초기화
CALL sys.ps_truncate_all_tables(FALSE);

-- 쿼리 프로파일링 활성화
UPDATE performance_schema.setup_instruments
SET enabled='YES', timed='YES'
WHERE name LIKE '%statement/%' OR name LIKE '%stage%';

UPDATE performance_schema.setup_consumers
SET enabled='YES'
WHERE name LIKE '%events_statements_%' OR name LIKE '%events_stages_%';

-- history_long 활성화 (기본 비활성)
UPDATE performance_schema.setup_consumers
SET ENABLED='YES'
WHERE NAME='events_statements_history_long';

-- 설정 관련 파라미터 (read only, restart 필요)
-- performance_schema_max_sql_text_length: SQL 텍스트 최대 저장 길이 (기본 1024)
-- performance_schema_max_digest_length: DIGEST 텍스트 최대 저장 길이 (기본 1024)
-- performance_schema_events_statements_history_size: 스레드당 히스토리 저장 수 (기본 10)
-- performance_schema_events_statements_history_long_size: history_long 전체 개수 (기본 10000)
```

---

### 접속·세션 모니터링

```sql
-- 계정/호스트별 접속 통계
SELECT * FROM performance_schema.accounts;
SELECT * FROM performance_schema.hosts
WHERE current_connections > 0 AND host NOT IN ('NULL', '127.0.0.1')
ORDER BY host;
SELECT * FROM performance_schema.users;

-- 미사용 DB 계정 확인
SELECT DISTINCT m_u.user, m_u.host
FROM mysql.user m_u
LEFT JOIN performance_schema.accounts ps_a ON m_u.user=ps_a.user AND ps_a.host=m_u.host
LEFT JOIN information_schema.views is_v ON is_v.definer=concat(m_u.user,'@',m_u.host) AND is_v.security_type='DEFINER'
LEFT JOIN information_schema.routines is_r ON is_r.definer=concat(m_u.user,'@',m_u.host) AND is_r.security_type='DEFINER'
WHERE ps_a.user IS NULL AND is_v.definer IS NULL AND is_r.definer IS NULL
ORDER BY m_u.user, m_u.host;

-- SHOW PROCESSLIST ID ↔ Thread ID 매핑
SELECT thread_id, processlist_id FROM performance_schema.threads WHERE processlist_id = 1507774;
SELECT sys.ps_thread_id(1507774);
```

---

### events_statements_summary_by_digest를 이용한 SQL 성능 분석

```sql
-- 1. 지연시간이 가장 긴 쿼리 (상위 5개)
SELECT IF(LENGTH(DIGEST_TEXT) > 64, CONCAT(LEFT(DIGEST_TEXT, 30), ' ... ', RIGHT(DIGEST_TEXT, 30)), DIGEST_TEXT) AS query,
       IF(SUM_NO_GOOD_INDEX_USED > 0 OR SUM_NO_INDEX_USED > 0, '*', '') AS full_scan,
       COUNT_STAR AS exec_count,
       SUM_ERRORS AS err_count,
       SUM_WARNINGS AS warn_count,
       SEC_TO_TIME(SUM_TIMER_WAIT/1000000000000) AS exec_time_total,
       SEC_TO_TIME(MAX_TIMER_WAIT/1000000000000) AS exec_time_max,
       (AVG_TIMER_WAIT/1000000000) AS exec_time_avg_ms,
       SUM_ROWS_SENT AS rows_sent,
       ROUND(SUM_ROWS_SENT/COUNT_STAR) AS rows_sent_avg,
       SUM_ROWS_EXAMINED AS rows_scanned
FROM performance_schema.events_statements_summary_by_digest
WHERE SCHEMA_NAME='DB명'
ORDER BY SUM_TIMER_WAIT DESC LIMIT 5;

-- 2. 임시 테이블 (정렬 최적화 확인)
-- disk_tmp_tables 사용률이 높으면 sort_buffer_size 증설 고려
SELECT IF(LENGTH(DIGEST_TEXT) > 64, CONCAT(LEFT(DIGEST_TEXT, 30), ' ... ', RIGHT(DIGEST_TEXT, 30)), DIGEST_TEXT) AS query,
       COUNT_STAR AS exec_count,
       SUM_CREATED_TMP_TABLES AS memory_tmp_tables,
       SUM_CREATED_TMP_DISK_TABLES AS disk_tmp_tables,
       ROUND(SUM_CREATED_TMP_DISK_TABLES/SUM_CREATED_TMP_TABLES*100) AS tmp_tables_to_disk_pct
FROM performance_schema.events_statements_summary_by_digest
WHERE SUM_CREATED_TMP_TABLES > 0 AND SCHEMA_NAME='DB명'
ORDER BY SUM_CREATED_TMP_DISK_TABLES DESC LIMIT 5;

-- 3. 인덱스 미사용 쿼리 (Full Scan)
SELECT IF(LENGTH(DIGEST_TEXT) > 64, CONCAT(LEFT(DIGEST_TEXT, 30), ' ... ', RIGHT(DIGEST_TEXT, 30)), DIGEST_TEXT) AS query,
       COUNT_STAR AS exec_count,
       SUM_NO_INDEX_USED AS no_index_used_count,
       ROUND((SUM_NO_INDEX_USED/COUNT_STAR)*100) AS no_index_used_pct
FROM performance_schema.events_statements_summary_by_digest
WHERE (SUM_NO_INDEX_USED > 0 OR SUM_NO_GOOD_INDEX_USED > 0) AND SCHEMA_NAME='DB명'
ORDER BY no_index_used_pct DESC, exec_count DESC LIMIT 5;

-- 4. 오류/경고 발생 쿼리
SELECT IF(LENGTH(DIGEST_TEXT)>64, CONCAT(LEFT(DIGEST_TEXT,30),' ... ',RIGHT(DIGEST_TEXT,30)), DIGEST_TEXT) AS query,
       COUNT_STAR AS exec_count,
       SUM_ERRORS AS errors, (SUM_ERRORS/COUNT_STAR)*100 AS error_pct,
       SUM_WARNINGS AS warnings
FROM performance_schema.events_statements_summary_by_digest
WHERE (SUM_ERRORS > 0 OR SUM_WARNINGS > 0) AND SCHEMA_NAME='DB명'
ORDER BY SUM_ERRORS DESC, SUM_WARNINGS DESC;
```

---

### sys 스키마를 활용한 성능 분석

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

-- 미사용 인덱스
SELECT * FROM sys.schema_unused_indexes WHERE object_schema='DB명';
-- Invisible로 변경 (안전하게 테스트)
ALTER TABLE 테이블명 ALTER INDEX 인덱스명 INVISIBLE;

-- 중복 인덱스
SELECT * FROM sys.schema_redundant_indexes \G;

-- 인덱스 통계 (선택/삽입/수정/삭제 횟수)
SELECT * FROM sys.schema_index_statistics LIMIT 10 \G;

-- 테이블별 I/O 및 변경량
SELECT table_schema, table_name, rows_fetched, rows_inserted, rows_updated, rows_deleted, io_read, io_write
FROM sys.schema_table_statistics
WHERE table_schema NOT IN ('mysql','performance_schema','sys');

-- Full scan 테이블
SELECT * FROM sys.schema_tables_with_full_table_scans;

-- I/O 요청 많은 파일
SELECT * FROM sys.io_global_by_file_by_bytes WHERE file LIKE '%ibd%';

-- auto_increment 사용률
SELECT table_schema, table_name, column_name,
       auto_increment AS current_value, max_value,
       ROUND(auto_increment_ratio*100, 2) AS usage_ratio
FROM sys.schema_auto_increment_columns;
```

---

### 쿼리 프로파일링 (Performance Schema 방식)

```sql
-- 1. 현재 설정 저장
CALL sys.ps_setup_save(10);

-- 2. 프로파일링 활성화
UPDATE performance_schema.setup_instruments SET enabled='YES', timed='YES'
WHERE name LIKE '%statement/%' OR name LIKE '%stage%';
UPDATE performance_schema.setup_consumers SET enabled='YES'
WHERE name LIKE '%events_statements_%' OR name LIKE '%events_stages_%';

-- 3. 쿼리 실행
SELECT * FROM tb_test1;

-- 4. EVENT_ID 확인
SELECT event_id, sql_text, sys.format_time(TIMER_WAIT) AS duration
FROM performance_schema.events_statements_history_long
WHERE SQL_TEXT LIKE '%tb_test1%';
-- event_id: 114, duration: 185.84 us

-- 5. 프로파일링 상세 확인
SELECT event_name AS stage, sys.format_time(TIMER_WAIT) AS duration
FROM performance_schema.events_stages_history_long
WHERE nesting_event_id=114    -- 위에서 확인한 EVENT_ID
ORDER BY timer_start;
-- stage/sql/starting → checking permissions → Opening tables → optimizing → executing ...

-- 6. 설정 복구
CALL sys.ps_setup_reload_saved();
```

---

### 메모리 사용 분석

```sql
-- MySQL 총 메모리 사용량
SELECT * FROM sys.memory_global_total;    -- 예: 16.46 GiB

-- 스레드별 메모리 사용량
SELECT thread_id, current_allocated FROM sys.memory_by_thread_by_current_bytes;

-- 모듈별 메모리 사용량
SELECT SUBSTRING_INDEX(event_name,'/',2) AS code_area,
       sys.format_bytes(SUM(current_alloc)) AS current_alloc
FROM sys.x$memory_global_by_current_bytes
GROUP BY SUBSTRING_INDEX(event_name,'/',2)
ORDER BY SUM(current_alloc) DESC;
-- memory/innodb: 6.27 GiB (가장 많음)

-- 특정 스레드 메모리 상세
SELECT thread_id, event_name, sys.format_bytes(CURRENT_NUMBER_OF_BYTES_USED) AS current_allocated
FROM performance_schema.memory_summary_by_thread_by_event_name
WHERE thread_id=46
ORDER BY CURRENT_NUMBER_OF_BYTES_USED DESC LIMIT 10;
```

---

### InnoDB 성능 튜닝 핵심

```sql
-- Redo Log 용량 조정 (8.0.31+)
SET GLOBAL innodb_redo_log_capacity = 12 * 1024 * 1024 * 1024;  -- 12GB

-- Buffer Pool 크기: 물리 메모리의 70~80%
-- buffer_pool_instances: CPU 코어 수/4 정도 (최대 8)
-- pool instance 증가 효과: throughput 증가보다 max latency 이상치 빈도 감소

-- 현재 InnoDB 파라미터 확인
SHOW VARIABLES LIKE 'innodb_buffer_pool%';
SHOW VARIABLES LIKE 'innodb_flush%';
SHOW VARIABLES LIKE 'innodb_log%';
SHOW ENGINE INNODB STATUS \G;
```

---

### Index 성능 최적화

**인덱스 최적화 방법**:
```sql
-- MySQL 8.0 인덱스 최적화 공식 문서
-- https://dev.mysql.com/doc/refman/8.0/en/optimization-indexes.html

-- 현재 인덱스 사용 횟수 확인
SELECT object_name, index_name, count_star
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE object_schema='DB명' AND object_name='테이블명';

-- 미사용 인덱스 확인
SELECT * FROM sys.schema_unused_indexes WHERE object_schema='DB명';
```

**fast index creation (`expand_fast_index_creation`)**:
```sql
-- 5억건 테이블 인덱스 재생성 속도 비교
-- OFF: drop primary 2시간 23분 / add primary 2시간 36분
-- ON:  drop primary 2시간 1분 / add primary 2시간 57분
-- → PK 재생성에는 효과 미미. Secondary index 추가/삭제에 효과적.

SET expand_fast_index_creation=ON;
SET SESSION sql_log_bin=OFF;     -- binlog 오버헤드 제거
ALTER TABLE 테이블명 ADD INDEX 인덱스명 (컬럼);
```

---

### Bulk Load 성능 최적화

대용량 데이터 적재 시 적용 순서:

```sql
-- 1. Binary Log OFF (세션)
SET SESSION sql_log_bin=OFF;

-- 2. Auto Commit OFF
SET autocommit=0;

-- 3. Constraint 체크 OFF
SET unique_checks=0;
SET foreign_key_checks=0;

-- 4. Multiple Insert 방식
INSERT INTO yourtable VALUES (1,2), (5,5), (3,4), ...;  -- 묶어서 한번에

-- 5. auto increment lock 모드 변경
SET innodb_autoinc_lock_mode=2;

-- 6. InnoDB 파라미터 임시 조정
SET GLOBAL innodb_flush_log_at_trx_commit=0;
SET GLOBAL innodb_log_buffer_size=64M;
SET GLOBAL sort_buffer_size=64M;

-- 7. LOAD DATA 사용 (가장 빠름 - INSERT 대비 20배 이상)
LOAD DATA INFILE '/tmp/data.csv' INTO TABLE 테이블명
FIELDS TERMINATED BY ',' LINES TERMINATED BY '\n';

-- 8. PK 순서로 Insert (clustered index 활용, random I/O 감소)

-- 9. 적재 완료 후 원복
SET autocommit=1;
SET unique_checks=1;
SET foreign_key_checks=1;
COMMIT;
SET SESSION sql_log_bin=ON;
SET GLOBAL innodb_flush_log_at_trx_commit=1;
```

---

### DB 용량·객체 조회

```sql
-- 데이터베이스별 용량
SELECT table_schema,
       SUM(data_length+index_length+data_free)/1024/1024 AS total_mb,
       SUM(data_length)/1024/1024 AS data_mb,
       COUNT(*) AS tables
FROM information_schema.tables
GROUP BY table_schema
ORDER BY 2 DESC;

-- PK 없는 테이블 조회
SELECT tab.table_schema AS database_name, tab.table_name
FROM information_schema.tables tab
LEFT JOIN information_schema.table_constraints tco
  ON tco.table_schema=tab.table_schema AND tco.table_name=tab.table_name
  AND tco.constraint_type='PRIMARY KEY'
WHERE tco.constraint_type IS NULL
  AND tab.table_schema NOT IN ('mysql','information_schema','performance_schema','sys')
  AND tab.table_type='BASE TABLE'
ORDER BY tab.table_schema, tab.table_name;

-- 인덱스 사용 횟수 조회
SELECT object_name, index_name, count_star
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE object_schema='DB명' AND object_name='테이블명';

-- 파티션 테이블별 용량
SELECT TABLE_SCHEMA, TABLE_NAME, PARTITION_NAME, TABLE_ROWS,
       ROUND(DATA_LENGTH/(1024*1024),2) AS 'DATA_SIZE(MB)'
FROM INFORMATION_SCHEMA.PARTITIONS
WHERE PARTITION_NAME IS NOT NULL AND TABLE_SCHEMA='DB명';
```

## 3. 연관 개념 (지식 연결)
- [[2026-06-12_(MySQL-Lock-Deadlock-모니터링)]] — Lock 모니터링 (performance_schema 활용)
- [[2026-06-12_(MySQL-관리자-쿼리-모음)]] — 관리자 운영 쿼리 모음
- [[2026-06-08_(Advanced-SQL-PostgreSQL)]] — PostgreSQL 성능 분석 비교 (EXPLAIN, pg_stat_statements)
