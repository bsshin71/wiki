# MySQL 쿼리 프로파일링 (SHOW PROFILES·EXPLAIN 칼럼)

- **카테고리**: #DBMS #쿼리튜닝
- **태그**: #쿼리튜닝 #MySQL #profiling #EXPLAIN #slow-query #분석
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-mysql 쿼리 profiling - BigDataTeam]]

## 1. 핵심 요약

`SET profiling=1`로 쿼리별 단계 실행 시간을 측정하고 `SHOW PROFILE FOR QUERY N`으로 상세 분석한다. `EXPLAIN`의 type·key·Extra 칼럼으로 인덱스 사용 여부와 정렬 방식을 파악한다.

---

## 2. 프로파일링 절차

```sql
SET profiling = 1;
SELECT * FROM store;          -- 분석할 쿼리 실행
SHOW PROFILES;                -- query ID 목록
SHOW PROFILE FOR QUERY 1;     -- 단계별 소요 시간 (Sending data, Sorting result 등)
```

## 3. EXPLAIN 주요 칼럼

| 칼럼 | 설명 |
|------|------|
| `id` | SELECT 구분 번호 |
| `select_type` | simple·primary·union·subquery·derived 등 |
| `type` | **const**(PK 단일행)·**eq_ref**(조인 PK)·**ref**(인덱스 참조)·**range**(범위)·**index**(인덱스 스캔)·**ALL**(전체 스캔) |
| `possible_keys` | 사용 가능한 인덱스 목록 |
| `key` | 실제 사용 인덱스 |
| `key_len` | 사용된 인덱스 길이 (복합 인덱스 몇 칼럼 사용 확인) |
| `rows` | 예상 검색 행 수 |
| `Extra` | **Using filesort**(정렬 추가), **Using temporary**(임시테이블), **Using index**(커버링 인덱스) 등 |

> ⚠️ `Extra: Using filesort` 또는 `Using temporary` → 성능 최적화 대상.

## 4. 연관 개념

- [[2026-06-14-QT02_(MySQL-실행계획-통계정보-히스토그램)]]
- [[2026-06-14-QT04_(MySQL-쿼리힌트-SKIP_SCAN-INDEX)]]
- [[2026-06-14-MS40_(MySQL-Percona-Toolkit-도구-모음)]] — pt-query-digest
- [[2026-06-13_(MySQL-Performance-Schema-활용)]]
