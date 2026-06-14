# PostgreSQL 문제 상황 (Index Bloating · Deadlock)

- **카테고리**: #DBMS #PostgreSQL #admin
- **태그**: #PostgreSQL #장애해결 #index_bloating #reindex #deadlock
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-문제 상황 시나리오 - BigDataTeam]]

## 1. 핵심 요약

**인덱스 블로팅**(예상보다 큰 인덱스 크기·과도한 스캔)은 `REINDEX` + `VACUUM ANALYZE`로 해소합니다.
**Deadlock**은 PostgreSQL이 자동 감지해 한 트랜잭션을 중단(`deadlock detected`)하여 해결하며, 서버 로그에서 관련 프로세스·쿼리를 확인합니다.

---

## 2. 인덱스 블로팅

### 확인
```sql
-- 인덱스 크기
SELECT indexrelname, pg_size_pretty(pg_relation_size(indexrelid))
FROM pg_stat_all_indexes WHERE schemaname='public';
-- 스캔 횟수(비정상적으로 높으면 블로팅 의심)
SELECT relname, indexrelname, idx_scan
FROM pg_stat_all_indexes WHERE schemaname='public' ORDER BY idx_scan DESC;
```

### 해결
```sql
REINDEX INDEX idx_name;     -- 특정 인덱스
REINDEX TABLE table_name;   -- 테이블의 모든 인덱스
VACUUM ANALYZE;             -- dead tuple 정리 + 통계 갱신
```

## 3. Deadlock

> PostgreSQL은 교착을 **자동 감지하고 한 트랜잭션을 중단(abort)** 해 나머지를 완료시킨다.

### 발생 예
```
-- T1: update customers set ... where idx=1;
-- T2: update ... where idx=2;  →  update ... where idx=1;  (T1 락 대기, 멈춤)
-- T1: update ... where idx=2;  →  ERROR: deadlock detected
```
```
ERROR:  deadlock detected
DETAIL:  Process 161472 waits for ShareLock on transaction 2598; blocked by process 159592.
         Process 159592 waits for ShareLock on transaction 2597; blocked by process 161472.
HINT:  See server log for query details.
```
> 중단된 트랜잭션만 롤백되고, 다른 트랜잭션은 정상 완료. 서버 로그에 양쪽 프로세스·STATEMENT 기록.

### 예방
- 트랜잭션에서 **객체 접근 순서를 일관**되게 유지(idx=1 → idx=2 순서 통일).
- 트랜잭션 짧게 유지, 필요한 행만 잠금.

## 4. 연관 개념

- [[2026-06-14-PG20_(PostgreSQL-상황별-모니터링-쿼리)]]
- [[2026-06-14-PG17_(PostgreSQL-AutoVacuum-설정-및-튜닝)]]
- [[2026-06-02_(PostgreSQL-EXPLAIN-실행계획-가이드)]]
