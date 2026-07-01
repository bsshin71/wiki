# Oracle SQL분석도구 — EXPLAIN PLAN & DBMS_XPLAN 실전 가이드
- **카테고리**: #Oracle #쿼리튜닝
- **태그**: #Oracle #쿼리튜닝 #EXPLAIN #PLAN_TABLE #DBMS_XPLAN #실행계획
- **작성일**: 2026-06-29
- **참조 원본**: [[2026-06-28_oracle_SQL분석도구_실행계획확인]]
- **is_public**: true

## 목차
- [[#1. 핵심 요약]]
- [[#2. PLAN_TABLE 준비]]
- [[#3. SQL*Plus에서 실행계획 확인]]
- [[#4. DBMS_XPLAN.DISPLAY 옵션]]
- [[#5. 연관 개념]]

## 1. 핵심 요약
Oracle SQL*Plus에서 EXPLAIN PLAN을 수행하고 `DBMS_XPLAN.DISPLAY`로 실행계획을 확인하는 실전 가이드. 10g 이상에서는 `PLAN_TABLE`이 자동 생성되어 별도 설치 불필요. `dbms_xplan.display`의 세 번째 인자(`advanced`, `serial`, `alias` 등)로 출력 상세도를 조절할 수 있다.

## 2. PLAN_TABLE 준비

### 10g 미만 버전 — 수동 생성 필요
```sql
SQL> @?/rdbms/admin/utlxplan.sql
```

### 10g 이상 버전 — 자동 존재 (별도 생성 불필요)
```sql
-- PUBLIC SYNONYM 확인
SELECT owner, synonym_name, table_owner, table_name
FROM all_synonyms
WHERE synonym_name = 'PLAN_TABLE';

-- OWNER    SYNONYM_NAME   TABLE_OWNER   TABLE_NAME
-- PUBLIC   PLAN_TABLE     SYS           PLAN_TABLE$
```

## 3. SQL*Plus에서 실행계획 확인

### EXPLAIN PLAN 수행
```sql
SQL> EXPLAIN PLAN FOR
  2  SELECT * FROM EMPLOYEES WHERE EMPLOYEE_ID=203;

Explained.
```

### 기본 실행계획 조회
```sql
SQL> SET LINESIZE 200
SQL> @?/rdbms/admin/utlxpls

PLAN_TABLE_OUTPUT
-----------------------------------------------------
Plan hash value: 1833546154

---------------------------------------------------------------------------------------------
| Id  | Operation                   | Name          | Rows  | Bytes | Cost (%CPU)| Time     |
---------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT            |               |     1 |    69 |     1   (0)| 00:00:01 |
|   1 |  TABLE ACCESS BY INDEX ROWID| EMPLOYEES     |     1 |    69 |     1   (0)| 00:00:01 |
|*  2 |   INDEX UNIQUE SCAN         | EMP_EMP_ID_PK |     1 |       |     0   (0)| 00:00:01 |
---------------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------
   2 - access("EMPLOYEE_ID"=203)
```

## 4. DBMS_XPLAN.DISPLAY 옵션

세 번째 인자로 출력 형식을 지정한다: `serial`, `parallel`, `outline`, `alias`, `projection`, `all`, `advanced`

### advanced 옵션 — 최상세 (Query Block, Outline, Predicate, Projection 포함)
```sql
SQL> SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY(NULL, NULL, 'advanced'));

-- Query Block Name / Object Alias (identified by operation id):
--    1 - SEL$1 / EMPLOYEES@SEL$1
--    2 - SEL$1 / EMPLOYEES@SEL$1
--
-- Outline Data
-- /*+
--     BEGIN_OUTLINE_DATA
--     INDEX_RS_ASC(@"SEL$1" "EMPLOYEES"@"SEL$1" ("EMPLOYEES"."EMPLOYEE_ID"))
--     OUTLINE_LEAF(@"SEL$1")
--     ALL_ROWS
--     DB_VERSION('19.1.0')
--     OPTIMIZER_FEATURES_ENABLE('19.1.0')
--     IGNORE_OPTIM_EMBEDDED_HINTS
--     END_OUTLINE_DATA
-- */
--
-- Column Projection Information (identified by operation id):
--    1 - "EMPLOYEE_ID"[NUMBER,22], "EMPLOYEES"."FIRST_NAME"[VARCHAR2,20], ...
--    2 - "EMPLOYEES".ROWID[ROWID,10], "EMPLOYEE_ID"[NUMBER,22]
```

### serial 옵션 — 직렬 실행 기본 정보
```sql
SQL> SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY(NULL, NULL, 'serial'));

-- 기본 Plan 테이블 + Predicate Information만 출력 (간결)
```

## 5. 연관 개념
- [[2026-06-15-OT02_(Oracle-실행계획-확인-방법-EXPLAIN-PLAN-DBMS_XPLAN)]]
- [[2026-06-15-OT03_(Oracle-실행계획-플랜-해석-방법)]]
- [[2026-06-15-OT04_(Oracle-19c-SQL-튜닝-절차-단계별-가이드)]]
- [[2026-06-29_(Oracle-SQL분석도구-AUTOTRACE-가이드)]]
- [[2026-06-30_(Oracle-SQL분석도구-SQL-Trace-TKPROF-가이드)]]
- [[2026-07-01_(Oracle-SQL분석도구-DBMS_XPLAN-심화-가이드)]]
