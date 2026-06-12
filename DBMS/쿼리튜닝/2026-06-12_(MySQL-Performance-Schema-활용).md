# MySQL Performance Schema 활용
- **카테고리**: #DBMS #쿼리튜닝
- **태그**: #MySQL #performance_schema #모니터링 #성능분석 #sys
- **작성일**: 2026-06-12
- **참조 원본**: [[2026-06-12-MySQL Performance_schema 활용 - BigDataTeam]], [[2026-06-12-Performance 스키마를 이용한 프로파일링 - BigDataTeam]], [[2026-06-12-events_statements_summary_by_digest를 이용하여 SQL성능 분석 - BigDataTeam]], [[2026-06-12-MySQL 성능 튜닝 - BigDataTeam]], [[2026-06-12-mysql optimization Index - BigDataTeam]], [[2026-06-12-fast index creation - BigDataTeam]], [[2026-06-12-성능 최적화를 위한 주요 커널파라미터 - BigDataTeam]], [[2026-06-12-데이터 Load 성능 올리기 - BigDataTeam]], [[2026-06-12-단순 Update문 지연사례 분석 - BigDataTeam]], [[2026-06-12-Update 문 지연2 - BigDataTeam]], [[2026-06-12-03. MySQL admin 쿼리 - BigDataTeam]], [[2026-06-12-mymon shell script for monitoring MySQL.md]]

## 1. 핵심 요약
- `sys` 스키마는 `performance_schema`의 복잡한 쿼리를 사용하기 쉽게 래핑한 뷰 모음. 실무에서 가장 먼저 사용.
- 느린 SQL 탐지: `sys.x$statement_analysis` (avg_latency 내림차순), 미사용 인덱스: `sys.schema_unused_indexes`.
- InnoDB 성능 튜닝 핵심: `innodb_buffer_pool_size` (물리 메모리 70~80%), `innodb_redo_log_capacity` (8.0.31+).

## 2. 상세 설명

### Performance Schema 초기화

```sql
-- 전체 테이블 초기화
CALL sys.ps_truncate_all_tables(FALSE);
```

---

### 접속·세션 모니터링

```sql
-- 계정/호스트별 접속 통계
SELECT * FROM performance_schema.accounts;
SELECT * FROM performance_schema.hosts WHERE current_connections > 0 AND host NOT IN ('NULL','127.0.0.1');

-- 미사용 DB 계정 확인
SELECT DISTINCT m_u.user, m_u.host
FROM mysql.user m_u
LEFT JOIN performance_schema.accounts ps_a ON m_u.user=ps_a.user AND ps_a.host=m_u.host
WHERE ps_a.user IS NULL
ORDER BY m_u.user, m_u.host;
```

---

### SQL 성능 분석

```sql
-- 실행시간 긴 쿼리 TOP 10
SELECT query, exec_count,
       sys.format_time(avg_latency) AS formatted_avg_latency,
       rows_sent_avg, rows_examined_avg, last_seen
FROM sys.x$statement_analysis
ORDER BY avg_latency DESC LIMIT 10;

-- Full Table Scan 발생 쿼리
SELECT db, query, exec_count,
       sys.format_time(total_latency) AS formatted_total_latency,
       rows_examined_avg, last_seen
FROM sys.x$statements_with_full_table_scans
WHERE db = 'your_db'
ORDER BY total_latency DESC LIMIT 10;

-- 자주 실행되는 쿼리
SELECT db, exec_count, query
FROM sys.statement_analysis
ORDER BY exec_count DESC LIMIT 10;

-- events_statements_summary_by_digest로 SQL 성능 분석
SELECT schema_name, digest_text, count_star, avg_timer_wait/1000000000 AS avg_ms,
       sum_rows_examined/count_star AS avg_rows_examined
FROM performance_schema.events_statements_summary_by_digest
WHERE schema_name = 'your_db'
ORDER BY avg_timer_wait DESC LIMIT 20;
```

---

### 인덱스 분석

```sql
-- 미사용 인덱스 확인
SELECT * FROM sys.schema_unused_indexes;

-- 인덱스를 INVISIBLE로 변경 (삭제 전 테스트)
ALTER TABLE 테이블명 ALTER INDEX 인덱스명 INVISIBLE;

-- 중복 인덱스 확인
SELECT * FROM sys.schema_redundant_indexes \G;

-- 인덱스 통계 (읽기/삽입/수정/삭제 횟수)
SELECT * FROM sys.schema_index_statistics LIMIT 10 \G;
```

---

### 메모리 사용 분석

```sql
-- MySQL 총 메모리 사용량
SELECT * FROM sys.memory_global_total;

-- 스레드별 메모리 사용량
SELECT thread_id, current_allocated FROM sys.memory_by_thread_by_current_bytes;

-- 모듈별 메모리 사용량
SELECT SUBSTRING_INDEX(event_name,'/',2) AS code_area,
       sys.format_bytes(SUM(current_alloc)) AS current_alloc
FROM sys.x$memory_global_by_current_bytes
GROUP BY SUBSTRING_INDEX(event_name,'/',2)
ORDER BY SUM(current_alloc) DESC;
```

---

### 테이블 I/O·변경량 분석

```sql
-- 변경 작업 많은 테이블
SELECT t.table_schema, t.table_name, tio.count_write
FROM information_schema.tables t
JOIN performance_schema.table_io_waits_summary_by_table tio
  ON tio.object_schema=t.table_schema AND tio.object_name=t.table_name
WHERE t.table_schema NOT IN ('mysql','performance_schema','sys')
ORDER BY tio.count_write DESC;

-- I/O 요청 많은 파일
SELECT * FROM sys.io_global_by_file_by_bytes WHERE file LIKE '%ibd%';
```

---

### InnoDB 성능 튜닝 핵심

```sql
-- InnoDB Redo Log 용량 조정 (8.0.31+)
SET GLOBAL innodb_redo_log_capacity = 12 * 1024 * 1024 * 1024;  -- 12GB

-- Buffer Pool 크기: 물리 메모리의 70~80%
-- buffer_pool_instances: 보통 CPU 코어 수 / 4 (최대 8)
-- pool instance 증가 시 throughput 증가보다 max latency 감소 효과
```

**성능 관련 파라미터 확인**:
```sql
SHOW VARIABLES LIKE 'innodb_buffer_pool%';
SHOW VARIABLES LIKE 'innodb_flush%';
SHOW ENGINE INNODB STATUS \G;  -- 전체 InnoDB 상태
```

---

### SHOW PROCESSLIST ↔ Thread ID 매핑

```sql
-- processlist_id로 thread_id 찾기
SELECT thread_id, processlist_id
FROM performance_schema.threads
WHERE processlist_id = 1507774;

-- 또는
SELECT sys.ps_thread_id(1507774);
```

---

### 트랜잭션 활성 세션의 쿼리 내역

```sql
SELECT ps_t.processlist_id, ps_esh.SQL_TEXT,
       sys.format_time(ps_esh.TIMER_WAIT) AS duration,
       date_sub(now(), interval (SELECT variable_value FROM performance_schema.global_status
                                 WHERE variable_name='UPTIME') - ps_esh.TIMER_START * 10e-13 second) AS start_time
FROM performance_schema.threads ps_t
INNER JOIN performance_schema.events_transactions_current ps_etc ON ps_etc.thread_id=ps_t.thread_id
INNER JOIN performance_schema.events_statements_history ps_esh ON ps_esh.NESTING_EVENT_ID=ps_etc.event_id
WHERE ps_etc.STATE='ACTIVE' AND ps_esh.MYSQL_ERRNO=0
ORDER BY ps_t.processlist_id, ps_esh.TIMER_START \G;
```

## 3. 연관 개념 (지식 연결)
- [[2026-06-12_(MySQL-Lock-Deadlock-모니터링)]] — Lock 모니터링 (performance_schema 활용)
- [[2026-06-12_(MySQL-관리자-쿼리-모음)]] — 관리자 운영 쿼리 모음
- [[2026-06-08_(Advanced-SQL-PostgreSQL)]] — PostgreSQL 성능 분석 비교
