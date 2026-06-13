# MySQL Update 지연 사례 — 디스크 커밋 지연 (Lock 아님)

- **카테고리**: #DBMS #MySQL #Lock
- **태그**: #lock #장애분석 #commit지연 #disk_io #waiting_for_handler_commit
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-단순 Update문 지연사례 분석 - BigDataTeam]]

## 1. 핵심 요약

지갑 DB의 단순 PK Update가 느려진 장애 사례로, 분석 결과 **레코드 락 대기가 아니라 디스크 커밋(I/O) 지연**이 원인이었습니다.
`sys.session`의 `state: waiting for handler commit`과 `data_locks` 전부 `GRANTED`(대기 없음)가 핵심 단서이며, `iostat %util` 포화로 다른 고부하 I/O가 커밋 flush를 지연시킨 것을 확인했습니다.

---

## 2. 증상 및 1차 점검 (PMM)

- 단순 PK Update가 평소보다 느림. 유입 트래픽·Transaction History·bulk write 변화 없음.
- PMM 특이점: **InnoDB Row Lock Wait Load / Time 증가** → 처음엔 락 의심.

---

## 3. Lock 위주 분석 → 락이 아님

### 3.1 commit 대기 세션
```sql
SELECT * FROM sys.session WHERE trx_state = 'ACTIVE'\G
-- state: waiting for handler commit   ← UPDATE는 끝났고 commit 단계 대기
-- statement_latency: 2.67 s, lock_latency: 0 ps
```

### 3.2 블로킹 조회 → 없음
```sql
-- data_lock_waits 조인 결과: No Result (대기 없음)
-- innodb_lock_waits: Empty set
```

### 3.3 data_locks → 전부 GRANTED
```sql
SELECT * FROM performance_schema.data_locks\G
-- 모든 행 LOCK_STATUS: GRANTED (IX 테이블락 + X,REC_NOT_GAP 레코드락)
```
> `GRANTED`만 존재 = **락 대기 세션 없음**. deadlock·row-lock contention 아님.

### 3.4 대상 쿼리 검증
```sql
-- PK(M_COIN_IDX, SERVER_IDX) 조건 단순 update, 테이블 15건 → 쿼리 자체 문제 없음
```

---

## 4. 지연 원인 판단 — 디스크 커밋 지연

`waiting for handler commit` = 레코드 업데이트는 완료, 그러나:
- **Redo Log / Binlog sync 미완료**
- **Group Commit / Flush 대기**
- 디스크 I/O 또는 커밋 처리 지연 (락 아님)

---

## 5. 디스크 지연 확인

```sql
SHOW ENGINE INNODB STATUS\G
-- "log flushes" 지연, "os file writes" 지연, "pending fsync" 많음 → 확정
```

```bash
iostat -x 1 10
vmstat 1 10
# await, svctm, %util 높음 → commit flush 지연
```

> 결론: `iostat %util 100%` — **다른 고부하 disk I/O 작업** 때문에 커밋 flush가 늦어진 것으로 판단.

---

## 6. 교훈

> "Row Lock Wait 지표 증가"라도 `data_locks`가 전부 GRANTED면 락 문제가 아니다.
> `waiting for handler commit` 상태는 **디스크/커밋 I/O 병목**을 의심해야 한다.

---

## 7. 연관 개념

- [[2026-06-13-57_(MySQL-Update-지연-사례-레코드락-commit-지연)]]
- [[2026-06-13-55_(MySQL-블로킹-세션-조회-누가-누구를-막는지)]]
- [[2026-06-13-26_(MySQL-Redo-Log-및-로그버퍼-설정)]]
- [[2026-06-13-52_(MySQL-오래-수행중인-TX-찾기)]]
