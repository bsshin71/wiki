# MySQL Lock · Deadlock 모니터링

- **카테고리**: #DBMS #MySQL #관리자 #모니터링
- **작성일**: 2026-06-13
- **참조 원본**: 
  - [[2026-06-12-mysql lock - BigDataTeam.md]]
  - [[2026-06-12-mysql deadlock - BigDataTeam.md]]
  - [[2026-06-12-Lock 개념 - BigDataTeam.md]]
  - [[2026-06-12-누가누구를 막고 있는지 보기 - BigDataTeam.md]]
  - [[2026-06-12-meta lock 을 잡고 있는 세션찾기 - BigDataTeam.md]]
  - [[2026-06-12-단순 Update문 지연사례 분석 - BigDataTeam.md]]
  - [[2026-06-12-Update 문 지연2 - BigDataTeam.md]]
  - [[2026-06-12-mysql commit 안하고 있는 세션찾기 - BigDataTeam.md]]

## 1. 핵심 요약

MySQL Lock 및 Deadlock 문제는 **대기 중인 트랜잭션 식별** → **원인 세션 특정** → **적절한 조치**의 3단계로 해결합니다.
주요 포인트는 DDL을 막는 **Metadata Lock (MDL_EXCLUSIVE)**와 DML 충돌의 **Record/Gap Lock (IX/X)**를 구분하는 것입니다.
Performance Schema 및 SYS 스키마를 활용하여 실시간 모니터링하고, 필요 시 세션을 종료합니다.

---

## 2. Lock 관련 주요 개념

### Lock의 분류

| 용어 | 의미 | 예시 |
|------|------|------|
| **Record Lock** | 실제 행(ROW)에 걸린 락 | `UPDATE user SET ... WHERE id=1;` |
| **Gap Lock** | 인덱스 간의 빈 공간을 보호 | `SELECT ... FOR UPDATE` 시 |
| **Next-Key Lock** | Record + Gap 결합형 (InnoDB의 기본) | 동시성 제어 기본값 |
| **Table Lock (MDL)** | 테이블 구조 관련 메타데이터 락 | `ALTER TABLE`, `CREATE INDEX` 등 |

### LOCK_MODE 및 LOCK_TYPE 해석

| LOCK_TYPE | LOCK_MODE | 의미 | 영향 범위 |
|-----------|-----------|------|----------|
| RECORD | X | 레코드 쓰기 잠금 (UPDATE/DELETE) | 행 단위 |
| RECORD | S | 레코드 읽기 잠금 (LOCK IN SHARE MODE) | 행 단위 |
| TABLE | IX | Intent Exclusive (DML 중) | 테이블 전체 |
| TABLE | IS | Intent Shared (SELECT 중) | 테이블 전체 |
| TABLE | X | 테이블 전체 쓰기 | 테이블 전체 |
| TABLE | S | 테이블 전체 읽기 | 테이블 전체 |
| TABLE | MDL_EXCLUSIVE | DDL 전용 (ALTER/DROP/TRUNCATE) | 테이블 전체 **[심각]** |
| TABLE | MDL_SHARED | DDL 읽기 공유 (SELECT) | 테이블 전체 |

> **❗ 핵심**: DDL(`ALTER`, `DROP`, `TRUNCATE`)이 안 되는 경우 대부분 **MDL_EXCLUSIVE** 또는 **해제되지 않은 IX 락** 때문입니다.

---

## 3. 실시간 Lock 모니터링 쿼리

### 3.1 전체 Lock 상태 한눈에 보기 (권장)

```sql
-- Performance Schema 기반 (MySQL 8.0+)
SELECT
  dl.OBJECT_SCHEMA,
  dl.OBJECT_NAME,
  dl.LOCK_TYPE,
  dl.LOCK_MODE,
  t.PROCESSLIST_ID AS pid,
  t.PROCESSLIST_USER AS user,
  t.PROCESSLIST_HOST AS host,
  es.SQL_TEXT AS current_sql
FROM performance_schema.data_locks dl
JOIN performance_schema.threads t ON t.thread_id = dl.thread_id
LEFT JOIN performance_schema.events_statements_current es ON es.thread_id = dl.thread_id
ORDER BY dl.OBJECT_NAME, dl.LOCK_MODE DESC;
```

### 3.2 특정 테이블의 Lock만 필터링

```sql
SELECT
  dl.OBJECT_SCHEMA,
  dl.OBJECT_NAME,
  dl.LOCK_TYPE,
  dl.LOCK_MODE,
  t.PROCESSLIST_ID AS pid,
  t.PROCESSLIST_USER AS user,
  t.PROCESSLIST_HOST AS host,
  es.SQL_TEXT AS current_sql
FROM performance_schema.data_locks dl
JOIN performance_schema.threads t ON t.thread_id = dl.thread_id
LEFT JOIN performance_schema.events_statements_current es ON es.thread_id = dl.thread_id
WHERE dl.OBJECT_SCHEMA = 'your_db'
  AND dl.OBJECT_NAME = 'your_table'
ORDER BY dl.LOCK_MODE DESC;
```

### 3.3 Lock 한방 조회 스크립트

```bash
#!/bin/bash
# MySQL Lock 종합 진단 스크립트

lock_check()
{
  mysql -uroot -p'PASSWORD' <<'EOF'
SELECT '====== 1. PROCESSLIST ======';
SHOW FULL PROCESSLIST;

SELECT '====== 2. LOCK 현황 ======';
SELECT
    ROLE,
    PROCESSLIST_ID,
    PROCESSLIST_USER,
    LOCK_TYPE,
    LOCK_MODE
FROM (
    SELECT
        t.THREAD_ID,
        CASE
            WHEN t.THREAD_ID = b.BLOCKING_THREAD_ID THEN 'GRANTER'
            WHEN t.THREAD_ID = b.REQUESTING_THREAD_ID THEN 'REQUESTER'
            ELSE 'UNKNOWN'
        END as ROLE,
        t.PROCESSLIST_ID,
        t.PROCESSLIST_USER,
        dl.LOCK_TYPE,
        dl.LOCK_MODE
    FROM performance_schema.threads t
    LEFT JOIN performance_schema.data_lock_waits b ON t.thread_id IN (b.blocking_thread_id, b.requesting_thread_id)
    LEFT JOIN performance_schema.data_locks dl ON dl.thread_id = t.thread_id
    WHERE dl.lock_type IS NOT NULL
) AS lock_info
ORDER BY PROCESSLIST_ID;

SELECT '====== 3. INNODB 트랜잭션 상태 ======';
SELECT * FROM information_schema.INNODB_TRX;
EOF
}

lock_check
```

---

## 4. Blocking 세션 식별 및 해결

### 4.1 누가 누구를 막고 있는지 확인 (권장)

```sql
-- MySQL 8.0+ 가장 간단한 방법: sys.schema_table_lock_waits
SELECT *
FROM sys.schema_table_lock_waits
ORDER BY waiting_pid;

-- 반환 컬럼:
-- - waiting_pid, blocking_pid
-- - waiting_query, blocking_query
-- - sql_kill_blocking_connection (자동 KILL 쿼리 생성)
```

### 4.2 상세 분석 (모든 정보 포함)

```sql
SELECT
  dlw.OBJECT_SCHEMA           AS waiting_schema,
  dlw.OBJECT_NAME             AS waiting_table,
  t_wait.PROCESSLIST_ID       AS waiting_pid,
  t_wait.PROCESSLIST_USER     AS waiting_user,
  es_wait.SQL_TEXT            AS waiting_sql,
  t_block.PROCESSLIST_ID      AS blocking_pid,
  t_block.PROCESSLIST_USER    AS blocking_user,
  es_block.SQL_TEXT           AS blocking_sql,
  dlw.LOCK_MODE               AS waiting_lock_mode,
  dlb.LOCK_MODE               AS blocking_lock_mode,
  TIMESTAMPDIFF(SECOND, t_block.PROCESSLIST_TIME, NOW()) AS blocking_duration_sec
FROM performance_schema.data_lock_waits w
JOIN performance_schema.data_locks dlw ON w.requesting_engine_lock_id = dlw.engine_lock_id
JOIN performance_schema.data_locks dlb ON w.blocking_engine_lock_id = dlb.engine_lock_id
JOIN performance_schema.threads t_wait ON t_wait.thread_id = dlw.thread_id
JOIN performance_schema.threads t_block ON t_block.thread_id = dlb.thread_id
LEFT JOIN performance_schema.events_statements_current es_wait ON es_wait.THREAD_ID = t_wait.thread_id
LEFT JOIN performance_schema.events_statements_current es_block ON es_block.THREAD_ID = t_block.thread_id
ORDER BY blocking_duration_sec DESC;
```

### 4.3 Blocking 세션 종료

```sql
-- 방법 1: 전체 세션 종료 (트랜잭션 ROLLBACK)
KILL <blocking_pid>;

-- 방법 2: 현재 쿼리만 취소 (트랜잭션은 유지)
KILL QUERY <blocking_pid>;

-- 방법 3: sys.schema_table_lock_waits 에서 자동 생성 쿼리 사용
-- (sql_kill_blocking_connection 컬럼 참고)
```

---

## 5. Metadata Lock (MDL) 진단

### 5.1 특정 테이블의 MDL 확인

```sql
-- 어떤 세션이 테이블에 Metadata Lock을 잡고 있는가?
SELECT
  OBJECT_TYPE,
  LOCK_TYPE,
  LOCK_STATUS,
  OWNER_THREAD_ID
FROM performance_schema.metadata_locks
WHERE OBJECT_SCHEMA = 'your_db'
  AND OBJECT_NAME = 'your_table'
ORDER BY OWNER_THREAD_ID;
```

### 5.2 MDL 보유 세션 조회

```sql
-- MDL 보유 세션의 상세 정보 (OWNER_THREAD_ID 이용)
SELECT
  t.PROCESSLIST_ID,
  t.PROCESSLIST_USER,
  t.PROCESSLIST_HOST,
  t.PROCESSLIST_DB,
  t.PROCESSLIST_COMMAND,
  t.PROCESSLIST_TIME,
  t.PROCESSLIST_STATE,
  t.PROCESSLIST_INFO
FROM performance_schema.threads t
WHERE t.THREAD_ID = <OWNER_THREAD_ID>  -- metadata_locks 에서 나온 값
ORDER BY PROCESSLIST_TIME DESC;
```

---

## 6. 트랜잭션 모니터링 (Commit 대기 세션)

### 6.1 Commit 안 하고 대기 중인 세션 찾기

```sql
-- 활성 트랜잭션 조회
SELECT
  trx_id,
  trx_state,
  trx_started,
  trx_mysql_thread_id,
  trx_query
FROM information_schema.innodb_trx
WHERE trx_state = 'RUNNING'
ORDER BY trx_started;  -- 가장 오래된 트랜잭션이 Lock 원인일 확률 높음

-- 또는 sys.session 이용 (더 간결함)
SELECT
  thd_id,
  conn_id,
  user,
  db,
  command,
  state,
  time,
  current_statement,
  trx_latency
FROM sys.session
WHERE trx_state = 'ACTIVE'
ORDER BY time DESC;
```

### 6.2 오래된 트랜잭션 (History Length) 모니터링

```sql
-- Undo Log History Length가 높으면 오래된 트랜잭션 존재
SELECT
  trx_id,
  trx_state,
  trx_started,
  TIMESTAMPDIFF(MINUTE, trx_started, NOW()) AS duration_min,
  trx_query
FROM information_schema.innodb_trx
WHERE TIMESTAMPDIFF(MINUTE, trx_started, NOW()) > 5  -- 5분 이상 열려있는 TX
ORDER BY trx_started;

-- Performance Schema 기반 (더 정교함)
SELECT
  t.THREAD_ID,
  t.PROCESSLIST_ID,
  t.PROCESSLIST_INFO,
  sys.format_time(ses.trx_latency) AS tx_duration
FROM performance_schema.threads t
LEFT JOIN sys.session ses ON ses.thd_id = t.THREAD_ID
WHERE ses.trx_state = 'ACTIVE'
ORDER BY ses.trx_latency DESC;
```

---

## 7. 실제 운영 사례: Update 지연 (Lock 아닌 Commit 대기)

### 배경

특정 테이블의 UPDATE가 평소보다 느리다는 보고. PMM 모니터링에서:
- InnoDB Row Lock Wait Load ↑
- InnoDB Row Lock Wait Time ↑

### 진단

```sql
SELECT * FROM sys.session WHERE trx_state = 'ACTIVE';
-- 결과: state = "waiting for handler commit" (중요!)
-- 의미: UPDATE/INSERT/DELETE 자체는 이미 끝났고, 커밋 단계에서 대기 중
```

### 원인 분석

- Lock Wait가 아니라 **Commit 단계의 대기**
- 여러 세션이 동시에 동일 테이블에 UPDATE 중
- InnoDB 커밋 엔진의 경합 (mutex contention)

### 해결 방법

1. **Commit을 기다리는 세션의 현재 SQL 확인**
   ```sql
   SELECT current_statement FROM sys.session WHERE trx_state = 'ACTIVE';
   ```

2. **장기 미완료 트랜잭션 롤백** (필요 시)
   ```sql
   KILL <blocking_pid>;
   ```

3. **Application 레벨 최적화**
   - Batch commit 간격 조정
   - autocommit=1 확인 (기본값: 활성화)
   - Batch Size 조정으로 커밋 빈도 감소

4. **DB 레벨 최적화**
   - `innodb_flush_log_at_trx_commit=2` (성능 향상, 안정성↓)
   - `sync_binlog=0` 임시 조정 (성능 향상, 복제 리스크↑)

---

## 8. Lock Wait Timeout 설정

### 조회

```sql
SHOW VARIABLES LIKE 'lock_wait_timeout';
-- 기본값: 31536000 초 (365일)
```

### 변경 (필요 시)

```sql
-- 세션 단위 변경
SET SESSION lock_wait_timeout = 30;

-- 글로벌 변경 (모든 신규 연결에 적용)
SET GLOBAL lock_wait_timeout = 30;

-- my.cnf 영구 설정
[mysqld]
lock_wait_timeout = 30
```

---

## 9. Performance Schema 활성화 확인

SQL_TEXT가 보이지 않으면 다음을 실행:

```sql
UPDATE performance_schema.setup_consumers
SET ENABLED='YES'
WHERE NAME IN (
  'events_statements_current',
  'events_statements_history',
  'events_statements_history_long'
);
```

---

## 10. 예방 가이드

| 조치 | 효과 | 우선순위 |
|------|------|---------|
| autocommit=1 유지 | 자동 트랜잭션 종료 | ⭐⭐⭐ |
| 주기적 innodb_trx 모니터링 | 오래된 TX 조기 발견 | ⭐⭐⭐ |
| Connection Pool Timeout 설정 | 좀비 연결 정리 | ⭐⭐ |
| 배치 작업 중 commit 간격 조정 | Commit 부하 분산 | ⭐⭐ |
| lock_wait_timeout 적절값 설정 | 무한 대기 방지 | ⭐⭐ |

---

## 11. 연관 개념

- [[2026-06-13_(MySQL-Performance-Schema-활용)]]
- [[2026-06-12_(MySQL-Replication-가이드)]]
- [[2026-06-02_(MySQL-데이터이관-Shell-스크립트)]]
