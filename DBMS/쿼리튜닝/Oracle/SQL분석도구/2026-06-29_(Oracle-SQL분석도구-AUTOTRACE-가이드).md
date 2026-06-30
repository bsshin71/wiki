# Oracle SQL분석도구 — AUTOTRACE 가이드
- **카테고리**: #Oracle #쿼리튜닝
- **태그**: #Oracle #쿼리튜닝 #AUTOTRACE #PLUSTRACE #실행계획
- **작성일**: 2026-06-29
- **참조 원본**: [[2026-06-29_oracle_SQL분석도구_Auto Trace]]
- **is_public**: true

## 목차
- [[#1. 핵심 요약]]
- [[#2. 권한 부여]]
- [[#3. 기본 사용법]]
- [[#4. 옵션 조합]]
- [[#5. 연관 개념]]

## 1. 핵심 요약
SQL*Plus의 `SET AUTOTRACE` 명령으로 SQL 실행 결과·예상 실행계획·실행 통계(Statistics)를 한 번에 확인하는 방법. `PLUSTRACE` 롤 및 `v_$session` 조회 권한이 없으면 `SP2-0618`/`SP2-0611` 오류가 발생하므로 sysdba로 사전 권한 부여가 필요하다. `on`/`traceonly`와 `explain`/`statistics` 옵션 조합으로 출력 범위를 제어할 수 있다.

## 2. 권한 부여
권한이 없으면 아래의 오류가 난다.
```
SQL> connect scott/tiger;
Connected.
SQL>  set autotrace on;
SP2-0618: Cannot find the Session Identifier.  Check PLUSTRACE role is enabled
SP2-0611: Error enabling STATISTICS report
```
오류시 아래의 권한을 sysdba role 로 부여한다.
```
[oracle@Rocky9 ~]$ sqlplus system/manager as sysdba
SQL> @?/sqlplus/admin/plustrce.sql
SQL> grant plustrace to scott;
SQL> grant select on v_$session to scott;
```

## 3. 기본 사용법
```
SQL> set autotrace on;
SQL> select * from scott.emp where empno =7900;

     EMPNO ENAME      JOB              MGR HIREDATE                  SAL       COMM     DEPTNO
---------- ---------- --------- ---------- ------------------ ---------- ---------- ----------
      7900 JAMES      CLERK           7698 03-DEC-81                 950                    30


Execution Plan
----------------------------------------------------------
Plan hash value: 2949544139

--------------------------------------------------------------------------------------
| Id  | Operation                   | Name   | Rows  | Bytes | Cost (%CPU)| Time     |
--------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT            |        |     1 |    87 |     1   (0)| 00:00:01 |
|   1 |  TABLE ACCESS BY INDEX ROWID| EMP    |     1 |    87 |     1   (0)| 00:00:01 |
|*  2 |   INDEX UNIQUE SCAN         | PK_EMP |     1 |       |     1   (0)| 00:00:01 |
--------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   2 - access("EMPNO"=7900)


Statistics
----------------------------------------------------------
         19  recursive calls
         17  db block gets
         12  consistent gets
          0  physical reads
       3072  redo size
        961  bytes sent via SQL*Net to client
        392  bytes received via SQL*Net from client
          1  SQL*Net roundtrips to/from client
          0  sorts (memory)
          0  sorts (disk)
          1  rows processed
```

## 4. 옵션 조합

| 번호  | 옵션                                     | 기능                                    |
| --- | -------------------------------------- | ------------------------------------- |
| 1   | set autotrace on                       | SQL 실행 , 결과 출력, 예상 실행계획 , 실행통계 출력<br> |
| 2   | set autotrace on explan [ exp ]        | SQL 실행,  결과 출력, 예상 실행계획               |
| 3   | set autotrace on statistics            | SQL 실행, 결과 출력, 실행 통계                  |
| 4   | set autotrace traceonly                | SQL 실행, 결과 출력(X), 예상 실행계획, 실행통계       |
| 5   | set autotrace traceonly explain[ exp ] | SQL 실행(X), 예상 실행계획                    |
| 6   | set autotrace traceonly statistics     | SQL 실행, 결과 출력(X), 실행통계                |

## 5. 연관 개념
- [[2026-06-29_(Oracle-SQL분석도구-EXPLAIN-PLAN-실행계획-가이드)]]
- [[2026-06-15-OT02_(Oracle-실행계획-확인-방법-EXPLAIN-PLAN-DBMS_XPLAN)]]
- [[2026-06-15-OT03_(Oracle-실행계획-플랜-해석-방법)]]
