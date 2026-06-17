# MySQL Metadata Lock Wait로 인한 Connection 폭주 패턴

- **카테고리**: #DBMS #MySQL #lock
- **태그**: #MySQL #lock #metadata-lock #connection폭주 #DDL #troubleshooting #long-transaction
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Waiting for table metadata lock 로 인한 connection 폭주 - BigDataTeam]]

## 1. 핵심 요약

**미 COMMIT 롱트랜잭션 세션 → DDL 대기 → 이후 SELECT 포함 모든 DML 메타락 대기**로 연쇄 폭주. 핵심은 **`kill -9` 등으로 후속 대기 세션을 죽여도 좀비처럼 살아 있다**는 점. **시발점(미commit 세션)을 찾아 kill**해야만 해소된다.

---

## 2. 폭주 패턴 (재현 시나리오)

| 단계 | session 1 | session 2 (DDL) | session 3 (SELECT) |
|------|-----------|-----------------|-------------------|
| 1 | `BEGIN; SELECT` (미commit) | — | — |
| 2 | — | `ALTER TABLE ... ADD COLUMN` → **MDL PENDING** | — |
| 3 | — | — | `SELECT` → **MDL PENDING** (session 2 때문) |
| 4 | — | — | `kill -9 session3_pid` → 프로세스 종료해도 **DB 세션 살아있음** |
| 5 | session1 kill → session2 kill 순서로 해소 | | |

> ⚠️ 대기 중인 session3를 먼저 죽여도 DB processlist에서 2670번이 그대로 남아 connection을 점유한다.

## 3. 시발점 찾기

```sql
-- 1) 60초 이상 활성 트랜잭션 조회
SELECT * FROM information_schema.innodb_trx
WHERE TIME_TO_SEC(TIMEDIFF(NOW(), trx_started)) > 60\G

-- 2) 해당 스레드의 실행 히스토리 (어떤 쿼리로 트랜잭션 열었는지)
SELECT is_pl.ID, ps_esh.SQL_TEXT, trx.trx_started,
  TIME_TO_SEC(TIMEDIFF(NOW(), trx.trx_started)) AS duration
FROM performance_schema.events_transactions_current ps_etc
JOIN information_schema.innodb_trx trx ON trx.trx_mysql_thread_id = ps_etc.THREAD_ID
JOIN performance_schema.events_statements_history ps_esh ON ps_esh.THREAD_ID = ps_etc.THREAD_ID
JOIN performance_schema.threads ps_th ON ps_th.THREAD_ID = ps_etc.THREAD_ID
JOIN information_schema.processlist is_pl ON is_pl.ID = ps_th.PROCESSLIST_ID
WHERE ps_etc.STATE = 'ACTIVE'
  AND TIME_TO_SEC(TIMEDIFF(NOW(), trx.trx_started)) > 50
ORDER BY ps_esh.THREAD_ID, ps_esh.EVENT_ID;

-- 3) 메타락 현황
SELECT object_name, lock_type, lock_status, processlist_id, processlist_info
FROM performance_schema.metadata_locks l
JOIN performance_schema.threads t ON t.thread_id = l.owner_thread_id
WHERE processlist_id <> connection_id();
```

## 4. 해결

```sql
KILL <시발점_process_id>;   -- 미commit 롱트랜잭션 세션 kill (session 1)
-- → DDL 세션(session 2)이 MDL 획득 → 완료 → 후속 대기 세션(session 3) 해소
```

> **핵심**: 대기열 맨 뒤(session 3)가 아닌 **가장 앞(session 1)**을 죽여야 한다.

## 5. 연관 개념

- [[2026-06-14-MS05_(MySQL-엔진-잠금-Global-Table-Named-Metadata)]] — MDL 개념
- [[2026-06-13-17_(MySQL-Meta-Lock-세션-조회)]]
- [[2026-06-14-MS03_(MySQL-InnoDB-Locking-Transaction-Model)]]
