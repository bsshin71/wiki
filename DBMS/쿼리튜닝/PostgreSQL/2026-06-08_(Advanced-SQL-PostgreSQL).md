# Advanced SQL — PostgreSQL 실무 가이드
- **카테고리**: #DBMS #쿼리튜닝 #PostgreSQL
- **태그**: #쿼리튜닝 #PostgreSQL #Oracle이행 #분석함수 #INDEX
- **작성일**: 2026-06-08
- **최종 재처리**: 2026-06-08 (`--confirm` 강제 재처리)
- **참조 원본**: [[Advanced_SQL_PG_20250325_v0.9]]

> Oracle 경험자를 위한 PostgreSQL Advanced SQL 실무 가이드. 원저: 류청하 / PG 번역·테스트: 임옥섭 (2026.03). 비공인 자료이며 기술적 검증이 필요한 부분이 포함될 수 있음.

---

## 1. 핵심 요약
- Oracle과 PostgreSQL의 SQL 문법·실행계획 차이를 실습 예제로 비교 정리한 중급 가이드.
- **쿼리 작성 시 주의**: 인덱스 무력화 패턴 5가지, 날짜 타입 차이, 정수 나눗셈 함정 등 Oracle 경험자가 자주 실수하는 포인트 중심.

---

## 2. 챕터별 핵심 정리

### Ch1. 기본 SELECT

| 포인트 | Oracle | PostgreSQL |
|--------|--------|------------|
| NULL 처리 | `NVL()` | `COALESCE()` (표준) |
| LIKE + 인덱스 | 기본 지원 | `text_pattern_ops` 옵션 또는 DB 생성 시 `COLLATION=C` 필요 |
| 함수 기반 인덱스(FBI) | 지원 | 지원 (`CREATE INDEX ON emp(REVERSE(ename))`) |
| 힌트 | `/*+ ... */` | `pg_hint_plan` 익스텐션 필요 (`/*+ IndexScan(t) */`) |

---

### Ch2. INDEX를 사용하는 SQL

**인덱스 무력화 5가지 패턴:**

| 패턴 | 예시 | 해결 |
|------|------|------|
| 조건식 부재/잘못된 표현 | `WHERE sal + 100 > 1000` | `WHERE sal > 900` |
| 컬럼 변형 | `WHERE TO_CHAR(hiredate,'YYYY') = '1981'` | `WHERE hiredate BETWEEN ...` |
| 부정형 비교 | `WHERE deptno != 10` | 가능하면 긍정형으로 재작성 |
| NULL 비교 | `WHERE comm = NULL` | `WHERE comm IS NULL` |
| 묵시적 형변환 | `WHERE empno = '7788'` (문자열) | 타입 일치 |

**실행계획 확인:**
```sql
EXPLAIN (ANALYZE, BUFFERS) SELECT ...;
```
- PG의 `Index Scan` = Oracle의 `INDEX RANGE SCAN`
- PG의 `Bitmap Heap Scan` = Oracle의 `INDEX SKIP SCAN`과 유사

**50만 건 테스트 데이터 생성 패턴:**
```sql
INSERT INTO emp ...
SELECT ... FROM emp e
CROSS JOIN generate_series(1, 100000) AS s(id)
LIMIT 500000 - 14;
ANALYZE emp; -- 통계정보 갱신
```

---

### Ch3. 날짜 함수

| 항목 | Oracle | PostgreSQL |
|------|--------|------------|
| DATE 타입 | 날짜+시간 포함 | **날짜만** (시간 없음) → `TIMESTAMP` 사용 |
| 날짜 포맷 출력 | `TO_CHAR(d, 'YYYY-MM-DD')` | 동일 |
| 문자→날짜 변환 | `TO_DATE()` | `TO_DATE()` / `TO_TIMESTAMP()` |
| 시스템 날짜 | `SYSDATE` | `CURRENT_DATE`, `NOW()`, `CURRENT_TIMESTAMP` |
| 날짜 연산 | `+ 1` (1일 추가) | `+ INTERVAL '1 day'` |

> ⚠️ Oracle의 DATE를 PG로 이행할 때 반드시 TIMESTAMP로 변경해야 한다.

---

### Ch4. TOP-n 질의

```sql
-- Oracle
SELECT ... FROM ... WHERE ROWNUM <= 10 ORDER BY sal DESC; -- ❌ 잘못된 패턴
SELECT * FROM (SELECT ... ORDER BY sal DESC) WHERE ROWNUM <= 10; -- ✅

-- PostgreSQL (표준 권장)
SELECT ... FROM ... ORDER BY sal DESC FETCH FIRST 10 ROWS ONLY;
-- 또는
SELECT ... FROM ... ORDER BY sal DESC LIMIT 10;
```

> ⚠️ `OFFSET` 사용 시 대용량 데이터에서 성능 저하 주의. 분석 함수(`RANK()`) 방식 권장.

---

### Ch5. 조인·서브쿼리

**조인 힌트 (pg_hint_plan):**
```sql
/*+ NestLoop(e d) */ -- Nested Loop 강제
/*+ HashJoin(e d) */ -- Hash Join 강제
/*+ MergeJoin(e d) */ -- Merge Join 강제
```

**Correlated Subquery vs JOIN 성능:**
- 소규모: Correlated Subquery도 무방
- 대규모: JOIN + 분석 함수로 재작성 권장

**LATERAL JOIN (PG 표준):**
```sql
SELECT ... FROM dept d, LATERAL (SELECT ... FROM emp e WHERE e.deptno = d.deptno LIMIT 2) t;
-- Oracle의 힌트 없이 NL 조인 유도 가능
```

**FULL OUTER JOIN:**
```sql
SELECT d.department_id, e.employee_id
FROM departments d
FULL OUTER JOIN employees e ON d.department_id = e.department_id;
```

---

### Ch6. WITH 절 (CTE)

```sql
WITH dept_avg AS (
    SELECT deptno, AVG(sal) avg_sal FROM emp GROUP BY deptno
)
SELECT e.ename, d.avg_sal FROM emp e JOIN dept_avg d ON e.deptno = d.deptno;
```

> ⚠️ **PG CTE 최적화 장벽(Optimization Fence)**: PG 12 이전에는 CTE가 항상 Materialized(물리화)되어 외부 조건이 CTE 내부로 전달되지 않음. PG 12+에서는 `WITH ... AS MATERIALIZED` / `AS NOT MATERIALIZED` 로 명시 제어 가능.

---

### Ch7. 그룹 함수

| 기능 | Oracle | PostgreSQL |
|------|--------|------------|
| 소계+합계 | `ROLLUP(a, b)` | 동일 |
| 모든 조합 | `CUBE(a, b)` | 동일 |
| 선택적 그룹 | `GROUPING SETS((a),(b),())` | 동일 |
| PIVOT | 비표준 `PIVOT` 구문 | `CASE + SUM` 또는 `crosstab()` (tablefunc) |
| UNPIVOT | 비표준 `UNPIVOT` 구문 | `UNION ALL` 또는 `unnest()` |

```sql
-- ROLLUP 예시
SELECT deptno, job, SUM(sal)
FROM emp
GROUP BY ROLLUP(deptno, job);
```

---

### Ch8. 분석 함수

**주요 함수:**

| 함수 | 용도 |
|------|------|
| `ROW_NUMBER() OVER(PARTITION BY ... ORDER BY ...)` | 중복 없는 순번 |
| `RANK()` | 동순위 허용, 다음 순위 건너뜀 |
| `DENSE_RANK()` | 동순위 허용, 다음 순위 연속 |
| `LAG(col, n)` / `LEAD(col, n)` | 이전/다음 행 값 참조 |
| `SUM() OVER(ORDER BY ...)` | 누적 합계 |
| `LISTAGG()` → `STRING_AGG(col, ',')` | 행→문자열 집계 |
| `RATIO_TO_REPORT()` | 비표준, PG 미지원 → `SUM() OVER()` 로 대체 |
| `CUME_DIST()` | 누적 분포 (0~1) |
| `PERCENT_RANK()` | 백분위 순위 (0~1) |

```sql
-- Inline View 대신 분석 함수로 성능 개선 패턴
SELECT ename, sal, deptno,
       AVG(sal) OVER(PARTITION BY deptno) AS dept_avg,
       sal - AVG(sal) OVER(PARTITION BY deptno) AS diff
FROM emp;
```

---

### Ch9. 계층 질의

Oracle의 `START WITH ... CONNECT BY`를 PG의 `WITH RECURSIVE`로 변환:

```sql
-- Oracle
SELECT empno, ename, LEVEL
FROM emp
START WITH mgr IS NULL
CONNECT BY PRIOR empno = mgr;

-- PostgreSQL (표준 Recursive CTE)
WITH RECURSIVE emp_hier AS (
    SELECT empno, ename, mgr, 1 AS lvl
    FROM emp WHERE mgr IS NULL
    UNION ALL
    SELECT e.empno, e.ename, e.mgr, h.lvl + 1
    FROM emp e JOIN emp_hier h ON e.mgr = h.empno
)
SELECT * FROM emp_hier;
```

| Oracle 구문 | PG 대안 |
|-------------|---------|
| `LEVEL` | `lvl` (CTE 내 계산 컬럼) |
| `CONNECT_BY_ROOT` | CTE에서 최상위 값 전파 |
| `SYS_CONNECT_BY_PATH` | CTE에서 문자열 누적 |
| `CONNECT_BY_ISLEAF` | 자식 없는 노드 판별 로직으로 대체 |
| `ORDER SIBLINGS BY` | `SEARCH DEPTH FIRST BY ... SET` (PG 표준) |

---

### Ch10. 정규식

```sql
-- 패턴 일치
SELECT * FROM t WHERE col ~ '정규식패턴';          -- 대소문자 구분
SELECT * FROM t WHERE col ~* '정규식패턴';         -- 대소문자 무시

-- 함수
REGEXP_REPLACE(col, '패턴', '치환문자')
REGEXP_MATCHES(col, '패턴', 'g')    -- 모든 매칭 반환
```

---

### 부록. Oracle → PostgreSQL 이행 가이드

**데이터 타입 매핑:**

| Oracle | PostgreSQL | 주의사항 |
|--------|------------|---------|
| `DATE` | `TIMESTAMP` | Oracle DATE는 시간 포함 |
| `NUMBER` | `NUMERIC` / `INT` / `BIGINT` | 소수점 없으면 정수형이 성능 유리 |
| `VARCHAR2` | `VARCHAR` / `TEXT` | PG의 TEXT는 길이 제한 없어 권장 |
| `CLOB/BLOB` | `TEXT` / `BYTEA` | TOAST 테이블로 별도 저장, VACUUM 필요 |
| `ROWID` | `ctid` (변경 가능) | PK 기반 로직으로 변경 권장 |

**주요 함수 변환:**

| Oracle | PostgreSQL |
|--------|------------|
| `NVL(a, b)` | `COALESCE(a, b)` |
| `DECODE(col, v1, r1, ...)` | `CASE WHEN ... END` |
| `SYSDATE` | `CURRENT_TIMESTAMP` / `NOW()` |
| `SEQ.NEXTVAL` | `nextval('seq')` 또는 `GENERATED ALWAYS AS IDENTITY` |
| `START WITH...CONNECT BY` | `WITH RECURSIVE` |
| `LISTAGG()` | `STRING_AGG()` |
| `PIVOT` | `CASE + SUM` / `crosstab()` |

**PG 특이사항:**

- **정수 나눗셈**: `1/3 = 0` (정수 결과) → `1/3.0` 또는 `1/3::numeric` 필요
- **트랜잭션 오류 처리**: SQL 오류 발생 시 자동 rollback 안 됨 → 반드시 `ROLLBACK` 명시 필요
- **CHAR(n)**: `n`은 바이트가 아닌 **문자 수**
- **테이블스페이스**: Oracle과 달리 물리 파일 상위 경로를 의미

---

## 3. 연관 개념 (지식 연결)
_(관련 문서가 생성되면 여기에 백링크를 추가합니다.)_
