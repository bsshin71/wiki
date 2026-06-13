# MySQL SQL 성능 분석 (Performance Schema)

- **카테고리**: #DBMS #MySQL #Performance
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-events_statements_summary_by_digest를 이용하여 SQL성능 분석 - BigDataTeam.md]]

## 1. 핵심 요약

Performance Schema의 `events_statements_summary_by_digest` 테이블은 **SQL 다이제스트 기준으로 집계된 성능 통계**를 제공합니다.
이를 통해 **느린 쿼리, 풀 스캔, 정렬 비효율, 인덱스 미사용** 문제를 식별하고,
정렬 메모리 부족, 임시 테이블 과다 사용 등의 병목을 분석할 수 있습니다.

---

## 2. Performance Schema 설정

### 2.1 필수 파라미터 (Read-Only, 재시작 필요)

| 파라미터 | 기본값 | 용도 |
|---------|-------|------|
| `performance_schema_max_sql_text_length` | 1024 | events_statements_summary_by_digest의 QUERY_SAMPLE_TEXT 최대 길이 |
| `performance_schema_max_digest_length` | 1024 | DIGEST_TEXT 최대 길이 |
| `performance_schema_events_statements_history_size` | 10 | 스레드당 히스토리 저장 개수 |
| `performance_schema_events_statements_history_long_size` | 10000 | 전체 롱 히스토리 저장 개수 |

**설정 확인:**
```sql
SHOW VARIABLES LIKE 'performance_schema%';
```

### 2.2 events_statements_history_long 활성화

```sql
-- 현재 상태 확인
SELECT * FROM performance_schema.setup_consumers 
WHERE NAME = 'events_statements_history_long';

-- 활성화 (필수)
UPDATE performance_schema.setup_consumers 
SET ENABLED = 'YES'
WHERE NAME = 'events_statements_history_long';
```

---

## 3. events_statements_summary_by_digest 주요 칼럼

| 칼럼 | 설명 |
|------|------|
| `DIGEST_TEXT` | 해시되지 않은 원본 SQL 다이제스트 |
| `COUNT_STAR` | 쿼리 누적 실행 횟수 |
| `SUM_TIMER_WAIT` | 총 대기 시간 (피코초 단위) |
| `AVG_TIMER_WAIT` | 평균 대기 시간 (피코초) |
| `MIN/MAX_TIMER_WAIT` | 최소/최대 대기 시간 |
| `SUM_ERRORS` | 누적 에러 발생 횟수 |
| `SUM_WARNINGS` | 누적 경고 발생 횟수 |
| `SUM_ROWS_SENT` | 반환된 총 행 수 |
| `SUM_ROWS_EXAMINED` | 검사한 총 행 수 |
| `SUM_NO_INDEX_USED` | 인덱스 미사용 횟수 |
| `SUM_NO_GOOD_INDEX_USED` | 부적절한 인덱스 사용 횟수 |
| `SUM_CREATED_TMP_TABLES` | 임시 테이블(메모리) 생성 횟수 |
| `SUM_CREATED_TMP_DISK_TABLES` | 임시 테이블(디스크) 생성 횟수 |
| `SUM_SORT_ROWS` | 정렬한 총 행 수 |
| `SUM_SORT_MERGE_PASSES` | 정렬 병합 패스 누적 횟수 |

---

## 4. 성능 분석 쿼리

### 4.1 느린 쿼리 TOP 5 (실행 시간 기준)

```sql
SELECT
  IF(LENGTH(DIGEST_TEXT) > 64, 
     CONCAT(LEFT(DIGEST_TEXT, 30), ' ... ', RIGHT(DIGEST_TEXT, 30)), 
     DIGEST_TEXT) AS query,
  IF(SUM_NO_GOOD_INDEX_USED > 0 OR SUM_NO_INDEX_USED > 0, '*', '') AS full_scan,
  COUNT_STAR AS exec_count,
  SUM_ERRORS AS err_count,
  SUM_WARNINGS AS warn_count,
  SEC_TO_TIME(SUM_TIMER_WAIT/1000000000000) AS total_time,
  SEC_TO_TIME(MAX_TIMER_WAIT/1000000000000) AS max_time,
  ROUND(AVG_TIMER_WAIT/1000000000, 2) AS avg_ms,
  SUM_ROWS_SENT AS rows_sent,
  ROUND(SUM_ROWS_SENT / COUNT_STAR) AS avg_rows,
  DIGEST
FROM performance_schema.events_statements_summary_by_digest
WHERE SCHEMA_NAME = 'your_db'
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 5;
```

**해석:**
- `full_scan='*'`: 풀 스캔 발생 중
- `avg_ms`: 평균 실행 시간 (밀리초)
- `avg_rows`: 쿼리당 반환 행 수

---

### 4.2 정렬 최적화 분석 (임시 테이블 사용)

```sql
SELECT
  IF(LENGTH(DIGEST_TEXT) > 64,
     CONCAT(LEFT(DIGEST_TEXT, 30), ' ... ', RIGHT(DIGEST_TEXT, 30)),
     DIGEST_TEXT) AS query,
  COUNT_STAR AS exec_count,
  SUM_CREATED_TMP_TABLES AS memory_tmp_tables,
  SUM_CREATED_TMP_DISK_TABLES AS disk_tmp_tables,
  ROUND(SUM_CREATED_TMP_TABLES / COUNT_STAR) AS avg_tmp_per_query,
  ROUND((SUM_CREATED_TMP_DISK_TABLES / SUM_CREATED_TMP_TABLES) * 100, 1) 
    AS tmp_to_disk_pct,
  DIGEST
FROM performance_schema.events_statements_summary_by_digest
WHERE SUM_CREATED_TMP_TABLES > 0
  AND SCHEMA_NAME = 'your_db'
ORDER BY SUM_CREATED_TMP_DISK_TABLES DESC, SUM_CREATED_TMP_TABLES DESC
LIMIT 5;
```

**해석:**
- `tmp_to_disk_pct`: 디스크 임시 테이블 비율
  - **낮을수록 좋음** (메모리에서 정렬)
  - **높으면** `sort_buffer_size` 증가 필요
- **대응책:**
  - `SET GLOBAL sort_buffer_size = 256M;` (기본 256K)
  - `SET GLOBAL tmp_table_size = 1G;`

---

### 4.3 정렬 성능 분석

```sql
SELECT
  IF(LENGTH(DIGEST_TEXT) > 64,
     CONCAT(LEFT(DIGEST_TEXT, 30), ' ... ', RIGHT(DIGEST_TEXT, 30)),
     DIGEST_TEXT) AS query,
  COUNT_STAR AS exec_count,
  SUM_SORT_MERGE_PASSES AS merge_passes,
  ROUND(SUM_SORT_MERGE_PASSES / COUNT_STAR) AS avg_merges,
  SUM_SORT_SCAN AS scans_using_scan,
  SUM_SORT_RANGE AS sorts_using_range,
  SUM_SORT_ROWS AS rows_sorted,
  ROUND(SUM_SORT_ROWS / COUNT_STAR) AS avg_rows_sorted,
  DIGEST
FROM performance_schema.events_statements_summary_by_digest
WHERE SUM_SORT_ROWS > 0
ORDER BY SUM_SORT_MERGE_PASSES DESC, SUM_SORT_SCAN DESC
LIMIT 5;
```

**해석:**
- `merge_passes`: 정렬 병합 패스 (낮을수록 좋음)
  - 높으면 정렬 메모리 부족
- `scans_using_scan`: 테이블 스캔으로 정렬
- `sorts_using_range`: 범위로 정렬 (더 효율적)
- **avg_rows_sorted > 10000**: 큰 데이터 정렬 주의

---

### 4.4 인덱스 성능 분석 (풀 스캔 감지)

```sql
SELECT
  IF(LENGTH(DIGEST_TEXT) > 64,
     CONCAT(LEFT(DIGEST_TEXT, 30), ' ... ', RIGHT(DIGEST_TEXT, 30)),
     DIGEST_TEXT) AS query,
  COUNT_STAR AS exec_count,
  SUM_NO_INDEX_USED AS no_index_count,
  SUM_NO_GOOD_INDEX_USED AS bad_index_count,
  ROUND((SUM_NO_INDEX_USED / COUNT_STAR) * 100, 1) AS no_index_pct,
  ROUND((SUM_NO_GOOD_INDEX_USED / COUNT_STAR) * 100, 1) AS bad_index_pct,
  DIGEST
FROM performance_schema.events_statements_summary_by_digest
WHERE (SUM_NO_INDEX_USED > 0 OR SUM_NO_GOOD_INDEX_USED > 0)
  AND SCHEMA_NAME = 'your_db'
ORDER BY no_index_pct DESC, exec_count DESC
LIMIT 5;
```

**해석:**
- `no_index_pct`: 풀 스캔 비율
  - **> 10%**: 인덱스 추가 검토 필요
- `bad_index_pct`: 부적절한 인덱스 사용
  - 존재하는 인덱스를 못 찾음 (쿼리 재작성 필요)
- **대응책:**
  - `ANALYZE TABLE table_name;`
  - 쿼리 HINT 추가: `USE INDEX (idx_name)`

---

### 4.5 에러/경고 발생 쿼리

```sql
SELECT
  IF(LENGTH(DIGEST_TEXT) > 64,
     CONCAT(LEFT(DIGEST_TEXT, 30), ' ... ', RIGHT(DIGEST_TEXT, 30)),
     DIGEST_TEXT) AS query,
  COUNT_STAR AS exec_count,
  SUM_ERRORS AS errors,
  ROUND((SUM_ERRORS / COUNT_STAR) * 100, 1) AS error_pct,
  SUM_WARNINGS AS warnings,
  ROUND((SUM_WARNINGS / COUNT_STAR) * 100, 1) AS warning_pct,
  DIGEST
FROM performance_schema.events_statements_summary_by_digest
WHERE (SUM_ERRORS > 0 OR SUM_WARNINGS > 0)
  AND SCHEMA_NAME = 'your_db'
ORDER BY SUM_ERRORS DESC, SUM_WARNINGS DESC;
```

**해석:**
- `error_pct`: 에러 발생 비율
  - **> 1%**: 쿼리 문제 심각
- `warning_pct`: 경고 발생 비율
  - 칼럼 타입 불일치, NULL 변환 등

---

## 5. 성능 진단 의사결정 트리

```
쿼리 느린가?
├─ 예 → 풀 스캔 있는가?
│   ├─ 예 → 인덱스 추가 또는 쿼리 재작성
│   └─ 아니오 → 임시 테이블 많은가?
│       ├─ 예 (디스크) → sort_buffer_size 증가
│       └─ 아니오 → JOIN/subquery 최적화
│
└─ 아니오
  └─ 테스트 완료
```

---

## 6. 주기적 모니터링 스크립트

```sql
-- 매 시간 실행 (cron job)
-- 느린 쿼리 자동 감지

CREATE EVENT IF NOT EXISTS perf_schema_monitor
ON SCHEDULE EVERY 1 HOUR
DO BEGIN
  INSERT INTO perf_analysis_log
  SELECT NOW(), * FROM (
    SELECT * FROM performance_schema.events_statements_summary_by_digest
    WHERE SCHEMA_NAME != 'mysql'
      AND SUM_TIMER_WAIT > 10000000000000  -- 10초 이상
    ORDER BY SUM_TIMER_WAIT DESC
    LIMIT 10
  ) AS recent_slow_queries;
END;
```

---

## 7. 체크리스트

| 항목 | 기준값 | 조치 |
|------|-------|------|
| 풀 스캔 비율 | < 5% | 인덱스 추가/최적화 |
| 평균 쿼리 시간 | < 100ms | 쿼리 재작성 |
| 디스크 임시 테이블 | < 20% | sort_buffer_size 증가 |
| 에러 비율 | < 1% | 쿼리 검증 |
| 정렬 병합 패스 | < 5 | 정렬 메모리 증가 |

---

## 8. 연관 개념

- [[2026-06-13-05_(MySQL-Admin-쿼리-기초)]]
- [[2026-06-13-09_(MySQL-fast-index-creation-최적화)]]
