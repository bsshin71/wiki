# MySQL Lock · Deadlock 모니터링
- **카테고리**: #DBMS #MySQL
- **태그**: #MySQL #lock #locking #모니터링 #troubleshooting
- **작성일**: 2026-06-12
- **참조 원본**: [[2026-06-12-Lock 개념 - BigDataTeam]], [[2026-06-12-mysql lock - BigDataTeam]], [[2026-06-12-mysql deadlock - BigDataTeam]], [[2026-06-12-누가누구를 막고 있는지 보기 - BigDataTeam]], [[2026-06-12-meta lock 을 잡고 있는 세션찾기 - BigDataTeam]], [[2026-06-12-단순 Update문 지연사례 분석 - BigDataTeam]], [[2026-06-12-Update 문 지연2 - BigDataTeam]]

## 1. 핵심 요약
- MySQL Lock 문제 진단의 핵심: **Metadata Lock(MDL) vs Data Lock** 구분. DDL이 막히면 MDL, DML끼리 충돌이면 Data Lock.
- `performance_schema.data_lock_waits`, `sys.innodb_lock_waits`, `sys.schema_table_lock_waits`를 상황에 맞게 사용.
- `LOCK_STATUS: GRANTED`이고 `state: waiting for handler commit`이면 **디스크 커밋 지연** (lock 문제가 아님).

## 2. 상세 설명

### Lock 주요 개념

| 용어 | 의미 |
|------|------|
| Record Lock | 실제 행(ROW)에 걸린 락 (`UPDATE ... WHERE id=1`) |
| Gap Lock | 인덱스 간 빈 공간 보호 (팬텀 방지) |
| Next-Key Lock | Record + Gap 결합형 (InnoDB 기본 행락) |
| Metadata Lock (MDL) | 테이블 구조 관련 락 (`ALTER`, `DROP`, `TRUNCATE`) |

### LOCK_MODE / LOCK_TYPE 해석

| LOCK_TYPE | LOCK_MODE | 의미 |
|-----------|-----------|------|
| RECORD | X | 레코드 쓰기 잠금 |
| RECORD | S | 레코드 읽기 잠금 |
| TABLE | IX | DML 실행 중 (Intent Exclusive) |
| TABLE | IS | SELECT 중 (Intent Shared) |
| TABLE | MDL_EXCLUSIVE | DDL 전용 락 |
| TABLE | MDL_SHARED | 읽기 공유 |

> DDL(`ALTER`, `DROP`)이 안 될 때 → 거의 항상 **MDL_EXCLUSIVE** 또는 **IX 락 미해제 세션** 때문

---

### 핵심 모니터링 쿼리

**Deadlock 발생 건수 확인**
```sql
SELECT name, count, status FROM INFORMATION_SCHEMA.INNODB_METRICS
WHERE name LIKE '%deadlocks%';
```

**누가 누구를 막고 있는지 (권장)**
```sql
SELECT
  dlw.OBJECT_NAME AS waiting_table,
  t_wait.PROCESSLIST_ID AS waiting_pid, t_wait.PROCESSLIST_USER AS waiting_user,
  es_wait.SQL_TEXT AS waiting_sql,
  t_block.PROCESSLIST_ID AS blocking_pid, t_block.PROCESSLIST_USER AS blocking_user,
  es_block.SQL_TEXT AS blocking_sql,
  CONCAT('KILL ', t_block.PROCESSLIST_ID, ';') AS kill_cmd
FROM performance_schema.data_lock_waits w
JOIN performance_schema.data_locks dlw ON w.REQUESTING_ENGINE_LOCK_ID = dlw.ENGINE_LOCK_ID
JOIN performance_schema.data_locks dlb ON w.BLOCKING_ENGINE_LOCK_ID = dlb.ENGINE_LOCK_ID
JOIN performance_schema.threads t_wait ON t_wait.thread_id = dlw.thread_id
JOIN performance_schema.threads t_block ON t_block.thread_id = dlb.thread_id
LEFT JOIN performance_schema.events_statements_current es_wait ON es_wait.THREAD_ID = t_wait.thread_id
LEFT JOIN performance_schema.events_statements_current es_block ON es_block.THREAD_ID = t_block.thread_id
ORDER BY blocking_pid, waiting_pid;
```

**Lock 한방 조회 Script**
```bash
lock_check() {
  mysql -uroot -pXXX <<EOF
SELECT 'show full processlist----';
SHOW FULL PROCESSLIST;
SELECT * FROM sys.innodb_lock_waits \G;
SELECT * FROM performance_schema.data_locks \G;
SELECT * FROM performance_schema.data_lock_waits \G;
SELECT * FROM information_schema.innodb_trx WHERE TIME_TO_SEC(TIMEDIFF(NOW(), trx_started)) > 1 \G;
SHOW ENGINE INNODB STATUS \G;
EOF
}
for i in {1..10}; do lock_check > $i.lock.data; done
```

**Metadata Lock 조회**
```sql
-- 특정 테이블 메타락 확인
SELECT OBJECT_TYPE, LOCK_TYPE, LOCK_STATUS, OWNER_THREAD_ID
FROM performance_schema.metadata_locks
WHERE OBJECT_SCHEMA='DB명' AND OBJECT_NAME='테이블명';

-- 메타락 잡은 세션 추적
SELECT t.PROCESSLIST_ID, t.PROCESSLIST_USER, t.PROCESSLIST_INFO
FROM performance_schema.threads t
WHERE t.THREAD_ID = <OWNER_THREAD_ID>;

-- 테이블 단위 잠금 대기 현황 (가장 간편)
SELECT * FROM sys.schema_table_lock_waits;
```

**commit 안 하고 대기 중인 세션 확인**
```sql
SELECT * FROM sys.session WHERE trx_state = 'ACTIVE';
```

**오래된 트랜잭션 (락 원인 후보)**
```sql
SELECT trx_id, trx_state, trx_started, trx_mysql_thread_id, trx_query
FROM information_schema.innodb_trx
WHERE trx_state = 'RUNNING'
ORDER BY trx_started;
```

---

### 장애 사례: Update 지연이 Lock이 아닌 경우

**증상**: slow query log에 Lock_time ≈ Query_time (20초)
**분석**:
1. `sys.innodb_lock_waits` → Empty (lock wait 없음)
2. `performance_schema.data_locks` → 전부 `LOCK_STATUS: GRANTED`
3. `sys.session` → `state: waiting for handler commit`

**결론**: **레코드 락이 아닌 디스크 커밋 지연**
- InnoDB UPDATE 자체는 완료, Redo Log/Binlog sync 단계에서 대기
- 확인: `SHOW ENGINE INNODB STATUS` → "log flushes" 지연, `iostat -x 1 10` → `%util 100%`

---

### Lock 해결 절차

```sql
-- 1. 원인 세션 식별
SELECT * FROM sys.schema_table_lock_waits;
-- 또는
SELECT * FROM sys.innodb_lock_waits;

-- 2. 세션 종료
KILL <blocking_pid>;          -- 연결 종료 (트랜잭션 롤백)
KILL QUERY <blocking_pid>;    -- 쿼리만 취소

-- 3. Lock timeout 조정 (임시)
SET GLOBAL lock_wait_timeout = 30;
```

## 3. 연관 개념 (지식 연결)
- [[2026-06-12_(MySQL-Performance-Schema-활용)]] — Performance Schema를 활용한 상세 모니터링
- [[2026-06-12_(MySQL-관리자-쿼리-모음)]] — commit 안 한 세션 찾기, 오래 수행 중인 TX 모음
