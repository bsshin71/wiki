# Oracle SQL분석도구 — DBMS_XPLAN 심화 가이드
- **카테고리**: #Oracle #쿼리튜닝
- **태그**: #Oracle #쿼리튜닝 #DBMS_XPLAN #display_cursor #실행계획 #ALLSTATS
- **작성일**: 2026-07-01
- **참조 원본**: [[2026-07-01_oracle SQL분석도구 DBMS_XPLAN]]
- **is_public**: true

## 목차
- [[#1. 핵심 요약]]
- [[#2. 특징]]
- [[#3. 예상 실행계획 출력]]
- [[#4. 캐싱된 커서의 실제 실행계획 출력]]
- [[#5. 캐싱된 커서의 Row Source 별 수행 통계 출력]]
- [[#6. 연관 개념]]

## 1. 핵심 요약
`dbms_xplan` 패키지의 세 가지 활용 시나리오:
① `display()` — PLAN_TABLE에 저장된 예상 실행계획 출력 (statement_id로 구분)
② `display_cursor()` — 라이브러리 캐시에 캐싱된 실제 실행계획 출력 (v$SQL의 sql_id·child_number 조회 필요)
③ `ALLSTATS`/`LAST` 포맷 — `gather_plan_statistics` 힌트와 함께 Row Source별 실제 수행 통계(A-Rows, A-Time, Buffers) 확인.

## 2. 특징
* Oracle 9.2 부터 dbms_xplan 패키지를 이용하여 plan_table 에 저장된 실행계획을 좀 더 쉽게 출력 가능하다.
* Oracle 10g 부터는 이 패키지로 라이브러리 캐시에 캐싱된 SQL 실행계획도 확인 가능하다.
* SQL 트레이스처럼 오퍼레이션 단계 (Row Source)별 수행통계도 쉽게 확인할 수 있다.

## 3. 예상 실행계획 출력

### statement_id 지정 후 쿼리 실행
```sql
SQL> explain plan set statement_id='SQL1' for
  2  select *
  3  from emp e, dept d
  4  where d.deptno = e.deptno
  5   and e.sal >= 1000
  6  ;
```

### plan 확인

#### 기본 옵션 (BASIC)
```sql
SQL> select * from table(dbms_xplan.display('PLAN_TABLE','SQL1','BASIC'));

PLAN_TABLE_OUTPUT
-------------------------------------------------------------------------------------
Plan hash value: 615168685

-----------------------------------
| Id  | Operation          | Name |
-----------------------------------
|   0 | SELECT STATEMENT   |      |
|   1 |  HASH JOIN         |      |
|   2 |   TABLE ACCESS FULL| DEPT |
|   3 |   TABLE ACCESS FULL| EMP  |
-----------------------------------
```
* 첫번째 인자 : plan table 명 지정
* 두번째 인자 : statement_id 입력. null 이면 가장 마지막 explain plan 명령의 실행계획 display
* 세번째 인자 : 포맷 옵션 지정

#### Row, Bytes, Cost 출력 옵션
```sql
SQL> select * from table(dbms_xplan.display('PLAN_TABLE','SQL1','BASIC ROWS BYTES COST'));

PLAN_TABLE_OUTPUT
----------------------------------------------------------------------------------------
Plan hash value: 615168685

----------------------------------------------------------------
| Id  | Operation          | Name | Rows  | Bytes | Cost (%CPU)|
----------------------------------------------------------------
|   0 | SELECT STATEMENT   |      |    12 |  1404 |     6   (0)|
|   1 |  HASH JOIN         |      |    12 |  1404 |     6   (0)|
|   2 |   TABLE ACCESS FULL| DEPT |     4 |   120 |     3   (0)|
|   3 |   TABLE ACCESS FULL| EMP  |    12 |  1044 |     3   (0)|
----------------------------------------------------------------

10 rows selected.
```

#### 추가 포맷 옵션 목록
* TYPICAL
* SERIAL
* PARTITION
* PARALLEL
* PREDICATE
* ALLIAS
* REMOTE
* NOTE
* ALL
* OUTLINE
* ADVANCED

## 4. 캐싱된 커서의 실제 실행계획 출력

라이브러리 캐시에 캐싱된 각 커서에 대한 통계는 `v$SQL` 뷰로, 실행계획은 `v$sql_plan` 뷰에서 확인 가능.

### 직전에 수행된 SQL의 sql_id와 child_number 확인
```sql
SQL> select prev_sql_id as sql_id , prev_child_number as child_no
  2  from v$session
  3  where sid=userenv('sid')
  4  and username is not null
  5  and prev_hash_value <> 0;

SQL_ID          CHILD_NO
------------- ----------
3xm3fd66dfsz4          0
```

### 더 이전에 수행된 쿼리 정보 확인
```sql
SQL>  select sql_id , child_number, sql_fulltext, last_active_time
  2  from v$sql
  3   where sql_text like  '%select%from%emp%dept%';

SQL_ID        CHILD_NUMBER SQL_FULLTEXT                               LAST_ACTIVE_TIME
------------- ------------ ------------------------------------------ ------------------
b9f0r3bsmtujb            0 explain plan set statement_id='SQL1' for    25-JUN-26
                           select *
                           from emp e, dept d
                           where d.dep

8jshd9nds3asu            0  select sql_id , child_number, sql_fulltext, 
                                  last_active_time                    25-JUN-26
                            from v$sql
                            where
```

### display_cursor 함수 이용
```sql
SQL> select * from  table(dbms_xplan.display_cursor('b9f0r3bsmtujb',0,'BASIC'));

PLAN_TABLE_OUTPUT
-----------------------------------------------------------------------------
EXPLAINED SQL STATEMENT:
------------------------
select sql_id , child_number, sql_fulltext, last_active_time from v$sql
where sql_text like  '%select%from%emp%dept%'

Plan hash value: 903671040

----------------------------------------------
| Id  | Operation        | Name              |
----------------------------------------------
|   0 | SELECT STATEMENT |                   |
|   1 |  FIXED TABLE FULL| X$KGLCURSOR_CHILD |
----------------------------------------------

14 rows selected.
```

직전 실행 SQL을 null로 조회하는 예시:
```sql
SQL> connect scott/tiger;
Connected.
SQL> set serveroutput off;
SQL> select * from emp;
-- (결과 14건 생략)

SQL> select * from table(dbms_xplan.display_cursor(null,null,'serial'));

PLAN_TABLE_OUTPUT
----------------------------------------------------------------------------------------
SQL_ID  a2dk8bdn0ujx7, child number 0
-------------------------------------
select * from emp

Plan hash value: 3956160932

--------------------------------------------------------------------------
| Id  | Operation         | Name | Rows  | Bytes | Cost (%CPU)| Time     |
--------------------------------------------------------------------------
|   0 | SELECT STATEMENT  |      |       |       |     3 (100)|          |
|   1 |  TABLE ACCESS FULL| EMP  |    14 |  1218 |     3   (0)| 00:00:01 |
--------------------------------------------------------------------------

17 rows selected.
```

### 필요한 권한
```sql
[oracle@Rocky9 ~]$ sqlplus sys/manager@devora as sysdba

SQL> grant select on v_$session to scott;
SQL> grant select on v_$sql to scott;
SQL> grant select on v_$sql_plan to scott;
```

## 5. 캐싱된 커서의 Row Source 별 수행 통계 출력

### 사용법
* 세션 레벨에서 `statistics_level` 을 all 로 설정
* 또는 SQL 문에 `gather_plan_statistics` 힌트 지정
* `grant select on v_$sql_plan_statistics_all to scott;` 권한 부여 후 조회

```sql
SQL> select /*+ gather_plan_statistics */ *
  2  from emp e, dept d
  3  where d.deptno = e.deptno
  4   and e.sal >= 1000;
  
SQL> select * from table(dbms_xplan.display_cursor(null,null,'ALLSTATS'));

LAN_TABLE_OUTPUT
---------------------------------------------------------------------------------------
SQL_ID  817t0q7cjk1c5, child number 0
-------------------------------------
select /*+ gather_plan_statistics */ * from emp e, dept d where
d.deptno = e.deptno  and e.sal >= 1000

Plan hash value: 615168685

-----------------------------------------------------------------------------------------
| Id  | Operation          | Name | Starts | E-Rows | A-Rows |   A-Time   | Buffers |  OMem |  1Mem |  O/1/M   |
-----------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT   |      |      1 |        |     12 |00:00:00.01 |      16 |       |       |          |
|*  1 |  HASH JOIN         |      |      1 |     12 |     12 |00:00:00.01 |      16 |  1399K|  1399K|     1/0/0|
|   2 |   TABLE ACCESS FULL| DEPT |      1 |      4 |      4 |00:00:00.01 |       7 |       |       |          |
|*  3 |   TABLE ACCESS FULL| EMP  |      1 |     12 |     12 |00:00:00.01 |       8 |       |       |          |
-----------------------------------------------------------------------------------------

Predicate Information (identified by operation id):
---------------------------------------------------

   1 - access("D"."DEPTNO"="E"."DEPTNO")
   3 - filter("E"."SAL">=1000)

Note
-----
   - dynamic statistics used: dynamic sampling (level=2)
```

* `Starts` : 각 오퍼레이션 단계를 몇 번 실행했는지
* `E-Rows` : 수행 전 옵티마이저가 예상한 로우 수

### DBMS_XPLAN ↔ SQL 트레이스 항목 대응표

| DBMS_XPLAN | SQL 트레이스            | 설명                         |     |
| ---------- | ------------------- | -------------------------- | --- |
| A-Rows     | rows                | 각 단계에서 읽거나 갱신한 건수          |     |
| A-Time     | time                | 각 단계별 소요시간                 |     |
| Buffers    | cr(=query), current | SQL 수행 과정에서 읽은 총 블록 수      |     |
| Reads      | pr                  | SQL 수행 과정에서 디스크에서 읽은 총 블록수 |     |

### LAST 옵션 — 마지막 수행 통계만 확인

각 항목은 기본적으로 누적값이므로, 마지막 수행 일량만 확인하려면 `LAST` 옵션을 추가한다.
```sql
select * from table(dbms_xplan.display_cursor(null,null,'ALLSTATS LAST'))
```

## 6. 연관 개념
- [[2026-06-29_(Oracle-SQL분석도구-EXPLAIN-PLAN-실행계획-가이드)]]
- [[2026-06-29_(Oracle-SQL분석도구-AUTOTRACE-가이드)]]
- [[2026-06-30_(Oracle-SQL분석도구-SQL-Trace-TKPROF-가이드)]]
- [[2026-06-15-OT02_(Oracle-실행계획-확인-방법-EXPLAIN-PLAN-DBMS_XPLAN)]]
