# Oracle 실행계획 확인 방법 (EXPLAIN PLAN · DBMS_XPLAN)

- **카테고리**: #DBMS #쿼리튜닝
- **태그**: #Oracle #쿼리튜닝 #실행계획 #EXPLAIN-PLAN #DBMS_XPLAN #SQL-Developer #Toad
- **작성일**: 2026-06-15
- **참조 원본**: [[2026-06-15-오라클 Sql 튜닝]]

## 목차
- [[#1. 핵심 요약]]
- [[#2. 방법 1 — EXPLAIN PLAN과 DBMS_XPLAN (정석)]]
- [[#3. 방법 2 — 툴(Tool) 활용 (가장 직관적)]]
- [[#4. 실행계획 해석 핵심 3가지]]
- [[#5. 연관 개념]]

## 1. 핵심 요약

오라클 실행계획 확인은 SQL 튜닝의 첫걸음이자 가장 중요한 단계입니다. 오라클이 내가 작성한 SQL을 어떤 경로로, 어떤 인덱스를 사용해 데이터를 가져오는지 확인하는 방법 3가지: ① EXPLAIN PLAN + DBMS_XPLAN (정석, SQL 미실행), ② DBMS_XPLAN.DISPLAY_CURSOR (실제 수행된 Real 계획), ③ SQL Developer(F10) / Toad(Ctrl+E) 단축키.

---

## 2. 방법 1 — EXPLAIN PLAN과 DBMS_XPLAN (정석)

SQL을 실제로 실행하지 않고, 오라클이 예상하는 최적의 실행 경로(통계 기반)만 빠르게 확인합니다. 데이터를 건드리지 않으므로 안전합니다.

### (1) 예상 실행계획 확인

```sql
-- Step 1: 실행계획 생성
EXPLAIN PLAN FOR
SELECT * FROM emp WHERE empno = 7788;

-- Step 2: 결과 조회
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
```

### (2) 실제 수행된 커서의 계획 확인 (Real 실행계획, 권장)

```sql
-- sql_id 찾기
SELECT sql_id, sql_text
FROM v$sql
WHERE sql_text LIKE '%SELECT * FROM emp%';

-- 실제 수행된 계획 + 실측 통계 확인
SELECT * FROM TABLE(
  DBMS_XPLAN.DISPLAY_CURSOR('여기_SQL_ID_입력', NULL, 'ALLSTATS LAST')
);
```

> ⭐ `EXPLAIN PLAN`은 "예상 계획"일 뿐, 실제 실행과 다를 수 있습니다. 실제 수행된 계획과 실측치를 봐야 정확한 튜닝이 가능합니다.

---

## 3. 방법 2 — 툴(Tool) 활용 (가장 직관적)

SQL Developer, Toad, Orange 같은 DB 관리 툴을 쓰고 있다면 명령어 없이 단축키 하나로 확인합니다.

| 툴 | 단축키 | 방법 |
|----|--------|------|
| **Oracle SQL Developer** | `F10` | 쿼리 창에 SQL 작성 후 F10 (또는 상단 돋보기 아이콘) |
| **Toad (토드)** | `Ctrl + E` | 쿼리 선택 후 Ctrl+E |

---

## 4. 실행계획 해석 핵심 3가지

결과 창이 뜨면 복잡한 표가 나와 당황스러울 수 있습니다. 핵심은 딱 3가지입니다.

### ① 계층형 구조 (실행 순서)

- **들여쓰기가 가장 깊은 곳(가장 안쪽 단계)**이 먼저 실행됩니다
- 들여쓰기 깊이가 같다면 **위에 있는 것**이 먼저 실행됩니다

### ② Operation (접근 방식)

| Operation | 의미 | 체크 포인트 |
|-----------|------|------------|
| `TABLE ACCESS FULL` | 테이블 전체를 다 읽음 | 데이터가 많다면 인덱스 미사용 요주의 대상 |
| `INDEX RANGE SCAN` | 인덱스를 범위로 잘 타고 있음 | 튜닝이 잘 된 형태 |
| `INDEX UNIQUE SCAN` | PK 등 유니크 인덱스로 단 1건 조회 | 가장 효율적 |

### ③ Cost / Rows (비용과 예상 건수)

- **Cost**: 오라클이 이 연산을 하기 위해 얼마나 힘들어하는지 (낮을수록 좋음)
- **Rows(Cardinality)**: 몇 건이나 나올지 예측한 값

이 수치가 비정상적으로 높은 구간이나, **예상 건수(E-Rows)와 실제 건수(A-Rows)의 괴리가 큰 구간**을 집중 공략해야 합니다.

---

## 5. 연관 개념

- [[2026-06-15-OT03_(Oracle-실행계획-플랜-해석-방법)]] — 실행계획 결과 표 상세 해석
- [[2026-06-15-OT04_(Oracle-19c-SQL-튜닝-절차-단계별-가이드)]] — 단계별 튜닝 절차
- [[2026-06-15-OT01_(Oracle-SQL-튜닝-실무-12단계-가이드)]] — 12단계 튜닝 가이드
- [[2026-06-02_(Oracle-쿼리튜닝-트러블슈팅)]]
