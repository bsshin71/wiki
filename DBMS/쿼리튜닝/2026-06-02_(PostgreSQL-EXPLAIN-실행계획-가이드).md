# PostgreSQL EXPLAIN 실행계획 가이드
- **카테고리**: #DBMS #쿼리튜닝
- **태그**: #PostgreSQL #쿼리튜닝 #EXPLAIN #실행계획
- **작성일**: 2026-06-02
- **참조 원본**: [[2026-06-02-PG explain 옵션]]

## 1. 핵심 요약
- PostgreSQL의 EXPLAIN 명령은 쿼리 실행 계획을 분석하는 핵심 도구. ANALYZE + BUFFERS 조합으로 실제 수행 시간과 캐시 사용 현황까지 확인 가능.

## 2. 상세 설명

### EXPLAIN 옵션 비교

| 명령어 | 실제 실행 | 실행 시간 | 버퍼 사용량 | 상세 정보 |
|--------|----------|----------|------------|----------|
| `EXPLAIN` | ❌ | ❌ | ❌ | 예상 실행 계획만 |
| `EXPLAIN (ANALYZE)` | ✅ | ✅ | ❌ | 실제 vs 예측 시간 비교 |
| `EXPLAIN (ANALYZE, BUFFERS)` | ✅ | ✅ | ✅ | 캐시(shared/local/temp) 사용량 |
| `EXPLAIN (ANALYZE, BUFFERS, VERBOSE)` | ✅ | ✅ | ✅ | 모든 출력 컬럼·내부 동작 상세 |

### 사용 예시

```sql
-- 예상 실행 계획만 확인 (실제 실행 안 함)
EXPLAIN SELECT * FROM employees WHERE department_id = 10;

-- 실제 실행 + 시간 측정
EXPLAIN (ANALYZE) SELECT * FROM employees WHERE department_id = 10;

-- 실제 실행 + 시간 + 캐시 사용량 (튜닝 시 권장)
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM employees WHERE department_id = 10;

-- 모든 정보 출력
EXPLAIN (ANALYZE, BUFFERS, VERBOSE) SELECT * FROM employees WHERE department_id = 10;
```

### 주요 실행 계획 노드

| 노드 | 설명 |
|------|------|
| `Seq Scan` | 전체 테이블 순차 스캔 |
| `Index Scan` | 인덱스를 통한 접근 (랜덤 I/O) |
| `Index Only Scan` | 인덱스만으로 결과 반환 (힙 접근 없음) |
| `Bitmap Heap Scan` | Bitmap Index Scan 후 힙 접근 |
| `Nested Loop` | 중첩 루프 조인 |
| `Hash Join` | 해시 조인 |
| `Merge Join` | 정렬 기반 병합 조인 |

### BUFFERS 해석

```
Buffers: shared hit=1234 read=56
```
- `shared hit`: 공유 버퍼(메모리 캐시)에서 읽은 블록 수
- `read`: 디스크에서 읽은 블록 수
- `hit` 비율이 높을수록 캐시 효율 좋음

### 힌트 사용 (pg_hint_plan 확장)

```sql
-- Seq Scan 강제
/*+ SeqScan(t) */ SELECT * FROM t WHERE ...

-- Index Scan 강제
/*+ IndexScan(t) */ SELECT * FROM t WHERE ...

-- 특정 조인 메서드 강제
/*+ NestLoop(a b) HashJoin(c d) */ SELECT ...
```

## 3. 연관 개념 (지식 연결)
- [[2026-06-08_(Advanced-SQL-PostgreSQL)]] — PostgreSQL Advanced SQL (EXPLAIN + 실행계획 심화)
- [[2026-06-02_(PostgreSQL-모니터링-쿼리-모음)]] — PostgreSQL 모니터링 쿼리
