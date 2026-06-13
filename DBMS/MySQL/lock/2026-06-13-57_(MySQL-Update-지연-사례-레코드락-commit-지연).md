# MySQL Update 지연 사례 — 레코드 락 + 앱 commit 지연

- **카테고리**: #DBMS #MySQL #Lock
- **태그**: #lock #장애분석 #record_lock #commit지연 #중복실행
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-Update 문 지연2 - BigDataTeam]]

## 1. 핵심 요약

slow log에 동일 행을 갱신하는 Update가 `Lock_time≈Query_time(약 20초)`로 다량 지연된 장애로, **레코드 락(X,REC_NOT_GAP) 대기**가 원인이었습니다.
근본 원인은 여러 서버에서 같은 `NOTI_TO_SERVICE` 행을 **중복 실행**하고, blocking 세션이 UPDATE 후 **앱(webhook) 코드에서 commit까지 지연**되며 락을 오래 쥐고 있었던 것입니다.

---

## 2. 증상 (slow log)

```
# Query_time: 19.95  Lock_time: 19.95  Rows_examined: 1  Rows_affected: 1
# InnoDB_rec_lock_wait: 19.95   Full_scan: No
UPDATE NOTI_TO_SERVICE SET STATUS=2, VERSION=VERSION+1
 WHERE NOTI_TO_SERVICE_IDX=496 AND VERSION=32;
```
- Query time 20초 ≈ **Lock time 19.9초** → 대부분 락 대기.
- 10.100.6.231, 10.100.6.232 **2개 서버에서 동시 실행**.
- rows_examined/affected = 1 → update 문 자체는 정상.

---

## 3. Lock waiting & blocking 수집

```sql
SELECT b.PROCESSLIST_ID AS waiting_process_id, b.PROCESSLIST_HOST AS waiting_host,
       b.PROCESSLIST_INFO AS waiting_query,
       a.PROCESSLIST_ID AS blocking_process_id, a.PROCESSLIST_INFO AS blocking_query,
       TIMESTAMPDIFF(SECOND, t.trx_started, NOW()) AS blocking_duration_sec,
       d.LOCK_MODE, d.LOCK_TYPE
FROM performance_schema.data_lock_waits c
  JOIN ... WHERE b.PROCESSLIST_INFO IS NOT NULL\G
-- waiting 60018 → blocking 60022, blocking_query=NULL,
-- blocking_duration_sec=27, LOCK_MODE=X,REC_NOT_GAP, LOCK_TYPE=RECORD
```

> `blocking_query=NULL` → blocking 세션은 쿼리 수행 후 **commit 대기 상태**. 레코드 IX/X 락으로 인한 대기.

---

## 4. commit 안 한 세션의 마지막 쿼리

```sql
SELECT * FROM sys.session WHERE trx_state = 'ACTIVE';
-- 60022: update NOTI_TO_SERVICE 수행 후 commit 대기 중
-- 50136: update NOTI_TO_SERVICE 수행 후 commit 대기 중
```

> `blocking_query`가 NULL이라 data_lock_waits만으로는 원인 쿼리를 못 찾음 → `sys.session`으로 **마지막 수행 쿼리**를 확인.

---

## 5. 결론

- `UPDATE NOTI_TO_SERVICE`가 **여러 서버에서 동일 행 중복 실행**.
- blocking 세션이 UPDATE 후 **앱의 다른 영역(webhook 코드 실행)에서 commit까지 지연** → 락을 장시간 점유.
- 개발자 코드 확인 결과 update 후 webhook 실행 단계에서 지연 가능성 확인.

> 교훈: blocking_query가 NULL(commit 대기)이면 **애플리케이션의 트랜잭션 경계(commit 시점)**를 의심하라.

---

## 6. 연관 개념

- [[2026-06-13-56_(MySQL-Update-지연-사례-디스크-커밋-지연)]]
- [[2026-06-13-55_(MySQL-블로킹-세션-조회-누가-누구를-막는지)]]
- [[2026-06-13-22_(MySQL-미커밋-세션-조회)]]
- [[2026-06-13-16_(MySQL-Lock-개념-및-분석)]]
