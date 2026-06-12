# MySQL Lock · Deadlock 모니터링
- **카테고리**: #DBMS #MySQL
- **태그**: #MySQL #lock #locking #deadlock #모니터링 #troubleshooting
- **작성일**: 2026-06-12
- **참조 원본**: [[2026-06-12-Lock 개념 - BigDataTeam]], [[2026-06-12-mysql lock - BigDataTeam]], [[2026-06-12-mysql deadlock - BigDataTeam]], [[2026-06-12-누가누구를 막고 있는지 보기 - BigDataTeam]], [[2026-06-12-meta lock 을 잡고 있는 세션찾기 - BigDataTeam]], [[2026-06-12-단순 Update문 지연사례 분석 - BigDataTeam]], [[2026-06-12-Update 문 지연2 - BigDataTeam]], [[2026-06-12-mysql commit 안하고 있는 세션찾기 - BigDataTeam]]

## 1. 핵심 요약
- MySQL Lock 문제 진단의 핵심: **Metadata Lock(MDL) vs Data Lock(Record/Gap/Next-Key)** 구분.
- `LOCK_STATUS: GRANTED`이면서 `state: waiting for handler commit`은 **디스크 커밋 지연**이며 Lock 문제가 아님.
- 핵심 모니터링 테이블: `performance_schema.data_lock_waits`, `sys.innodb_lock_waits`, `sys.schema_table_lock_waits`, `information_schema.innodb_trx`

## 2. 상세 설명

### Lock 주요 개념

| 용어 | 의미 | 발생 상황 |
|------|------|-----------|
| **Record Lock** | 실제 행(ROW)에 걸린 락 | `UPDATE ... WHERE id=1` |
| **Gap Lock** | 인덱스 간 빈 공간 보호 (팬텀 방지) | `SELECT ... FOR UPDATE` |
| **Next-Key Lock** | Record + Gap 결합 (InnoDB 기본) | 동시성 제어 |
| **MDL (Metadata Lock)** | 테이블 구조 관련 락 | `ALTER TABLE`, `DROP`, `TRUNCATE` |
| **IX** | Intent Exclusive (DML 실행 중) | |
| **IS** | Intent Shared (SELECT 중) | |

> 💡 DDL(`ALTER`, `DROP`, `TRUNCATE`)이 안 될 때 → 거의 항상 **MDL_EXCLUSIVE 또는 IX 락 미해제 세션** 때문

### LOCK_MODE / LOCK_TYPE 해석

| LOCK_TYPE | LOCK_MODE | 의미 |
|-----------|-----------|------|
| RECORD | X | 레코드 쓰기 잠금 (UPDATE 등) |
| RECORD | S | 레코드 읽기 잠금 (LOCK IN SHARE MODE) |
| TABLE | IX | DML 실행 중 (Intent Exclusive) |
| TABLE | IS | SELECT 중 (Intent Shared) |
| TABLE | MDL_EXCLUSIVE | DDL 전용 락 |
| TABLE | MDL_SHARED | 읽기 공유 |

---

### Deadlock 모니터링

```sql
-- Deadlock 발생 건수 확인
SELECT name, count, status FROM INFORMATION_SCHEMA.INNODB_METRICS
WHERE name LIKE '%deadlocks%';

-- 전체 InnoDB 상태 (SHOW ENGINE INNODB STATUS의 deadlock 섹션)
SHOW ENGINE INNODB STATUS \G;
```

---

### Lock 한방 조회 Script

```bash
lock_check() {
  mysql -uroot -pXXX <<EOF
SELECT 'show full processlist----';
SHOW FULL PROCESSLIST;
SELECT 'sys.innodb_lock_waits---';
SELECT * FROM sys.innodb_lock_waits \G;
SELECT 'performance_schema.data_locks---';
SELECT * FROM performance_schema.data_locks \G;
SELECT 'performance_schema.data_lock_waits---';
SELECT * FROM performance_schema.data_lock_waits \G;
SELECT 'innodb_trx (>1 sec)---';
SELECT * FROM information_schema.innodb_trx WHERE TIME_TO_SEC(timediff(now(),trx_started)) > 1 \G;
SELECT 'show engine innodb status---';
SHOW ENGINE INNODB STATUS \G;
EOF
}
for i in {1..10}; do lock_check > $i.lock.data; done
```

---

### 누가 누구를 막고 있는지 (권장 쿼리)

```sql
-- MySQL 8.0+ (권장)
SELECT
  dlw.OBJECT_NAME AS waiting_table,
  t_wait.PROCESSLIST_ID AS waiting_pid,
  t_wait.PROCESSLIST_USER AS waiting_user,
  es_wait.SQL_TEXT AS waiting_sql,
  t_block.PROCESSLIST_ID AS blocking_pid,
  t_block.PROCESSLIST_USER AS blocking_user,
  es_block.SQL_TEXT AS blocking_sql,
  dlw.LOCK_TYPE AS waiting_lock_type,
  dlw.LOCK_MODE AS waiting_lock_mode,
  CONCAT('KILL ', t_block.PROCESSLIST_ID, ';') AS kill_cmd
FROM performance_schema.data_lock_waits w
JOIN performance_schema.data_locks dlw ON w.REQUESTING_ENGINE_LOCK_ID = dlw.ENGINE_LOCK_ID
JOIN performance_schema.data_locks dlb ON w.BLOCKING_ENGINE_LOCK_ID = dlb.ENGINE_LOCK_ID
JOIN performance_schema.threads t_wait ON t_wait.thread_id = dlw.thread_id
JOIN performance_schema.threads t_block ON t_block.thread_id = dlb.thread_id
LEFT JOIN performance_schema.events_statements_current es_wait ON es_wait.THREAD_ID = t_wait.thread_id
LEFT JOIN performance_schema.events_statements_current es_block ON es_block.THREAD_ID = t_block.thread_id
ORDER BY blocking_pid, waiting_pid;

-- 간단 버전 (sys schema)
SELECT * FROM sys.innodb_lock_waits;
SELECT * FROM sys.schema_table_lock_waits;
```

---

### Metadata Lock (MDL) 조회

```sql
-- 특정 테이블의 메타락 정보
SELECT OBJECT_TYPE, LOCK_TYPE, LOCK_STATUS, OWNER_THREAD_ID
FROM performance_schema.metadata_locks
WHERE OBJECT_SCHEMA = 'DB명' AND OBJECT_NAME = '테이블명';

-- 메타락 잡은 세션 상세 확인
SELECT t.PROCESSLIST_ID, t.PROCESSLIST_USER, t.PROCESSLIST_HOST,
       t.PROCESSLIST_TIME, t.PROCESSLIST_STATE, t.PROCESSLIST_INFO
FROM performance_schema.threads t
WHERE t.THREAD_ID = <OWNER_THREAD_ID>;

-- alter 문 대기하게 만든 세션의 전체 쿼리 이력 확인
SELECT ps_t.processlist_id, concat(ps_t.PROCESSLIST_USER,'@',ps_t.PROCESSLIST_HOST) AS db_account,
       ps_esh.event_name, ps_esh.SQL_TEXT,
       sys.format_time(ps_esh.TIMER_WAIT) AS duration,
       date_sub(now(), interval (SELECT variable_value FROM performance_schema.global_status
                                 WHERE variable_name='UPTIME') - ps_esh.TIMER_START*10e-13 second) AS start_time
FROM performance_schema.events_statements_history ps_esh
INNER JOIN performance_schema.threads ps_t ON ps_t.thread_id = ps_esh.thread_id
WHERE ps_t.processlist_id = <blocking_pid>    -- show processlist의 id 입력
  AND ps_esh.SQL_TEXT IS NOT NULL
  AND ps_esh.MYSQL_ERRNO = 0
ORDER BY ps_esh.TIMER_START \G;
```

---

### commit 안 하고 대기 중인 세션 찾기

```sql
-- 현재 active 트랜잭션 세션 (sys.session)
SELECT * FROM sys.session WHERE db='CFD' AND trx_state = 'ACTIVE' \G;

-- 60초 이상 commit 안 한 트랜잭션
SELECT * FROM information_schema.innodb_trx
WHERE TIME_TO_SEC(timediff(now(), trx_started)) > 60 \G;

-- Lock 대기 관계 (lock granter/requester + 최근 쿼리이력)
SELECT
       a.ROLE, a.PROCESSLIST_ID, a.user,
       sys.format_time(b.TIMER_WAIT) AS TIMER_WAIT,
       sys.format_time(b.LOCK_TIME) AS LOCK_TIME,
       b.start_time,
       ifnull(a.PROCESSLIST_INFO, b.SQL_TEXT) AS query,
       c.LOCK_TYPE, c.LOCK_MODE
FROM
    (SELECT a.THREAD_ID,
            CASE WHEN a.THREAD_ID = b.BLOCKING_THREAD_ID THEN 'GRANTER'
                 WHEN a.THREAD_ID = b.REQUESTING_THREAD_ID THEN 'REQUESTER'
                 ELSE 'UNKNOW' END AS ROLE,
            a.PROCESSLIST_ID, a.PROCESSLIST_INFO
     FROM performance_schema.threads a, performance_schema.data_lock_waits b
    ) a,
    performance_schema.data_locks c
    LEFT OUTER JOIN LATERAL (
        SELECT h.THREAD_ID, h.SQL_TEXT, h.TIMER_WAIT, h.LOCK_TIME,
               date_sub(now(), interval (SELECT variable_value FROM performance_schema.global_status
                                         WHERE variable_name='UPTIME') - h.TIMER_START*10e-13 second) AS start_time
        FROM performance_schema.events_statements_history h
        WHERE h.THREAD_ID = c.THREAD_ID
        ORDER BY h.EVENT_ID DESC LIMIT 1
    ) b ON c.thread_id = b.thread_id
WHERE a.THREAD_ID = c.THREAD_ID;
```

---

### 오래된 트랜잭션 (History Length) 모니터링

```sql
-- transaction history length 확인
SELECT count FROM INFORMATION_SCHEMA.INNODB_METRICS
WHERE NAME = 'trx_rseg_history_len';

-- 장시간 수행중인 DML (10초 이상 executing 상태)
SELECT b.processlist_user, b.processlist_state,
       SEC_TO_TIME(a.TIMER_WAIT/1000000000000) AS timer_wait,
       IF(LENGTH(a.SQL_TEXT) > 64, CONCAT(LEFT(a.SQL_TEXT, 30), '...', RIGHT(a.SQL_TEXT, 30)), a.SQL_TEXT) AS query,
       a.ROWS_AFFECTED, a.ROWS_SENT, a.ROWS_EXAMINED
FROM performance_schema.events_statements_current a,
     performance_schema.threads b
WHERE a.current_schema = 'DB명'
  AND a.thread_id = b.thread_id
  AND b.processlist_state = 'executing'
ORDER BY a.timer_wait DESC LIMIT 10;

-- 최근 10분 이내 10초 이상 수행되었던 쿼리 이력
SELECT ps_t.THREAD_ID, concat(ps_t.PROCESSLIST_USER,'@',ps_t.PROCESSLIST_HOST) AS db_account,
       ps_esh.SQL_TEXT, sys.format_time(ps_esh.TIMER_WAIT) AS duration,
       date_sub(now(), interval (SELECT variable_value FROM performance_schema.global_status
                                 WHERE variable_name='UPTIME') - ps_esh.TIMER_START*10e-13 second) AS start_time
FROM performance_schema.events_statements_history ps_esh
LEFT OUTER JOIN performance_schema.threads ps_t ON ps_esh.THREAD_ID = ps_t.THREAD_ID
WHERE (SELECT variable_value FROM performance_schema.global_status WHERE variable_name='UPTIME') - ps_esh.TIMER_START*10e-13 < 10*60
  AND ps_esh.TIMER_WAIT*10e-13 > 10
ORDER BY TIMER_START, ps_esh.TIMER_WAIT DESC;
```

---

### 장애 사례 분석: Update 지연이 Lock이 아닌 경우

**증상**: slow query log에 `Lock_time ≈ Query_time` (20초)
**분석 과정**:
```sql
-- 1. lock wait 확인
SELECT * FROM sys.innodb_lock_waits;              -- Empty set → lock wait 없음

-- 2. data_locks 확인
SELECT * FROM performance_schema.data_locks \G;
-- 전부 LOCK_STATUS: GRANTED → 대기 세션 없음

-- 3. 현재 세션 상태
SELECT * FROM sys.session WHERE trx_state = 'ACTIVE';
-- state: waiting for handler commit
```

**결론**:
- `LOCK_STATUS: GRANTED`, `state: waiting for handler commit` → **레코드 락 대기가 아님**
- 원인: **InnoDB UPDATE 완료 → Redo Log / Binlog sync 단계에서 대기**
- **디스크 커밋 지연** (디스크 I/O 병목)

**확인 방법**:
```bash
# InnoDB 상태에서 "log flushes" 지연, "pending fsync" 확인
SHOW ENGINE INNODB STATUS \G;

# OS 디스크 I/O 확인
iostat -x 1 10
vmstat 1 10
# %util이 100%이면 디스크 I/O 병목
```

---

### Lock 해결 절차

```sql
-- 1. 원인 세션 식별
SELECT * FROM sys.schema_table_lock_waits;    -- MDL 관련
SELECT * FROM sys.innodb_lock_waits;           -- Row lock 관련

-- 2. processlist에서 세션 상태 확인
SHOW FULL PROCESSLIST;

-- 3. 세션 종료
KILL <blocking_pid>;          -- 연결 종료 (트랜잭션 롤백)
KILL QUERY <blocking_pid>;    -- 쿼리만 취소

-- 4. Lock timeout 설정 (임시 완화)
SHOW VARIABLES LIKE 'lock_wait_timeout';
SET GLOBAL lock_wait_timeout = 30;   -- 기본값 31536000 (1년)
SET GLOBAL innodb_lock_wait_timeout = 50;  -- InnoDB row lock timeout

-- 5. 장기 오픈 트랜잭션 예방
-- autocommit=1 기본 유지
-- 주기적으로 information_schema.innodb_trx 모니터링
```

## 3. 연관 개념 (지식 연결)
- [[2026-06-12_(MySQL-Performance-Schema-활용)]] — Performance Schema 상세 모니터링 쿼리
- [[2026-06-12_(MySQL-관리자-쿼리-모음)]] — 관리자 운영 쿼리 모음 (오래 수행 TX, 커밋 안한 세션)
