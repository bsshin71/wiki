# MySQL InnoDB 스토리지엔진 잠금 (레코드·갭·넥스트키)

- **카테고리**: #DBMS #MySQL #lock
- **태그**: #MySQL #lock #innodb #record-lock #gap-lock #next-key #lock-wait
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-InnoDB 스토리지 엔진 잠금 - BigDataTeam]]

## 1. 핵심 요약

InnoDB는 MySQL 서버 락과 별개로 **레코드 기반 잠금**을 가진다. 특징: **락 에스컬레이션 없음**, **갭락 존재**. 레코드락은 레코드가 아니라 **인덱스의 레코드**를 잠그며, PK/유니크 변경은 GAP 없이 레코드만(REC_NOT_GAP), 보조 인덱스 변경은 Next-Key/Gap을 사용한다. **바이너리 로그를 ROW 포맷**으로 바꾸면 락 발생이 줄어든다. 이 문서는 실무 Lock 대기 조회 쿼리 중심.

---

## 2. 잠금 종류 (도서 기반 정리)

- **레코드락**: 인덱스 레코드 잠금. PK/UK → REC_NOT_GAP, 보조 인덱스 → Next-Key/Gap.
- **갭락**: 레코드 사이 간격 잠금 → 그 사이 INSERT 제어.
- **넥스트 키 락**: 레코드락+갭락. Deadlock/Lock wait 자주 유발 → **binlog ROW 포맷**으로 완화.

## 3. Lock 대기 순서 조회 쿼리 (대기↔블로킹)

```sql
SELECT
  r.trx_id waiting_trx_id, r.trx_mysql_thread_id waiting_thread, r.trx_query waiting_query,
  b.trx_id blocking_trx_id, b.trx_mysql_thread_id blocking_thread, b.trx_query blocking_query
FROM performance_schema.data_lock_waits w
JOIN information_schema.innodb_trx b ON b.trx_id = w.blocking_engine_transaction_id
JOIN information_schema.innodb_trx r ON r.trx_id = w.requesting_engine_transaction_id;
```

## 4. 스레드별 잠금 상세

```sql
SELECT * FROM performance_schema.data_locks
WHERE ENGINE_TRANSACTION_ID = 152022 \G;
-- 예: TABLE IX (GRANTED) + RECORD X,REC_NOT_GAP (PK index, LOCK_DATA=100001)
```
> PK 인덱스로 레코드 락 → `REC_NOT_GAP`(갭 미포함). 레거시 `INNODB_LOCKS/INNODB_LOCK_WAITS`도 사용 가능(8.0은 `performance_schema.data_locks` 권장).

## 5. 연관 개념

- [[2026-06-14-MS03_(MySQL-InnoDB-Locking-Transaction-Model)]] — 락/격리수준 개념 레퍼런스
- [[2026-06-13-24_(MySQL-InnoDB-Lock-모니터링)]] · [[2026-06-13-55_(MySQL-블로킹-세션-조회-누가-누구를-막는지)]]
