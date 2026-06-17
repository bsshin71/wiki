# MySQL 쿼리힌트 (SKIP_SCAN·INDEX·STRAIGHT_JOIN)

- **카테고리**: #DBMS #쿼리튜닝
- **태그**: #쿼리튜닝 #MySQL #쿼리힌트 #SKIP_SCAN #INDEX_HINT #EXPLAIN #optimizer-hint
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-MySQL 쿼리힌트 - BigDataTeam]]

## 1. 핵심 요약

MySQL 옵티마이저가 최적 플랜을 선택하지 못할 때 **힌트**로 유도한다. 대표적: `SKIP_SCAN`으로 복합 인덱스 선두 칼럼 미사용 시 인덱스 활용, `IGNORE INDEX`로 불필요한 인덱스 제외, `STRAIGHT_JOIN`으로 조인 순서 고정. `EXPLAIN FORMAT=TREE`·`EXPLAIN ANALYZE`로 실제 실행 비용 확인.

---

## 2. EXPLAIN FORMAT=TREE / ANALYZE

```sql
EXPLAIN FORMAT=TREE
SELECT * FROM employees e
  IGNORE INDEX(PRIMARY, ix_hiredate)
  INNER JOIN dept_emp de IGNORE INDEX(ix_empno_fromdate, ix_fromdate)
  ON de.emp_no = e.emp_no AND de.from_date = e.hire_date\G;
-- → 내부 해시 조인·비용·rows 트리 구조로 출력
```

## 3. SKIP_SCAN 힌트

복합 인덱스 `(gender, birth_date)`에서 `birth_date`만 조건 시 인덱스 미사용 → `SKIP_SCAN`으로 강제:
```sql
EXPLAIN SELECT /*+ SKIP_SCAN(employees ix_gender_birthdate) */ COUNT(*)
FROM employees WHERE birth_date >= '1965-02-01';
-- type: range, key: ix_gender_birthdate, Extra: Using index for skip scan
```

## 4. 주요 인덱스 힌트

| 힌트 | 의미 |
|------|------|
| `USE INDEX(idx)` | 해당 인덱스 사용 권고 |
| `FORCE INDEX(idx)` | 해당 인덱스 강제 사용 |
| `IGNORE INDEX(idx)` | 해당 인덱스 제외 |
| `STRAIGHT_JOIN` | 쿼리에 나온 순서로 조인 고정 |

## 5. Optimizer Hint (/*+ ... */) 주요 목록

```sql
/*+ BNL(t1, t2) */       -- Block Nested Loop 조인
/*+ NO_BNL(t1, t2) */
/*+ HASH_JOIN(t1, t2) */
/*+ INDEX(t idx) */
/*+ SKIP_SCAN(t idx) */
/*+ SEMIJOIN(FIRSTMATCH) */
```

## 6. 연관 개념

- [[2026-06-14-QT02_(MySQL-실행계획-통계정보-히스토그램)]]
- [[2026-06-14-QT03_(MySQL-쿼리-프로파일링-EXPLAIN-칼럼)]]
- [[2026-06-14-QT05_(MySQL-튜닝사례-지연된조인-래터럴조인)]]
