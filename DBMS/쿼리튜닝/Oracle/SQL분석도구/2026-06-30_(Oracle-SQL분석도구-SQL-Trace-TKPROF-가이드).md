﻿# Oracle SQL분석도구 — SQL Trace & TKPROF 가이드
- **카테고리**: #Oracle #쿼리튜닝
- **태그**: #Oracle #쿼리튜닝 #SQL_TRACE #TKPROF #실행계획
- **작성일**: 2026-06-30
- **참조 원본**: [[2026-06-30_oracle_SQL분석도구_SQL트레이스]]
- **is_public**: true

## 목차
- [[#1. 핵심 요약]]
- [[#2. 소개]]
- [[#3. 권한설정]]
- [[#4. 세션에 trace 설정]]
- [[#5. 트레이스 파일 위치 확인]]
- [[#6. 리포트 생성]]
- [[#7. 트레이스 결과분석]]
- [[#8. 연관 개념]]

## 1. 핵심 요약
사전 실행계획·AUTOTRACE만으로 원인을 찾기 어려울 때 사용하는 SQL 튜닝 최후 도구. `alter session set sql_trace=true`로 세션 단위 trace 파일을 생성하고, `v$diag_info`로 경로를 확인한 뒤 `tkprof`로 사람이 읽기 쉬운 리포트로 변환한다. 리포트의 Parse/Execute/Fetch call 통계와 AUTOTRACE 항목(`current`/`query`/`disk` ↔ `db block gets`/`consistent gets`/`physical reads`) 대응 관계를 알아야 두 도구를 비교 해석할 수 있다.

## 2. 소개
* SQL 튜닝할때 가장 많이 사용하는 도구
* 사전 실행계획과 AutoTrace 결과만으로 문제를 찾을 수 없을 때 사용

## 3. 권한설정
```
$sqlplus system/manager
SQL> grant alter session to scott;
```

## 4. 세션에 trace 설정
```
SQL> alter session set sql_trace=true;

SQL> select * from emp where empno = 7900;
SQL> select * from dual;

SQL> alter session set sql_trace=false;
```

## 5. 트레이스 파일 위치 확인
### 경로 확인
```
SQL> select value
  2  from v$diag_info
  3  where name = 'Diag Trace';

VALUE
-------------------------------------------------------
/oracle/u01/app/oracle/diag/rdbms/devdb/devdb/trace
```

### 경로 및 파일명 확인
```
SQL> select value
  2  from v$diag_info
  3  where name = 'Default Trace File';

VALUE
---------------------------------------------------------------------------------------------------------
/oracle/u01/app/oracle/diag/rdbms/devdb/devdb/trace/devdb_ora_6096.trc

```

## 6. 리포트 생성
트레이스 파일 내용 그대로 분석하기는 쉽지 않아 TKProf 유틸리티를 사용한다.
트레이스 파일을 보기 쉽게 포맷팅한 리포트를 생성해 준다.
### tkprof 사용법
```
Usage: tkprof tracefile outputfile [explain= ] [table= ]
              [print= ] [insert= ] [sys= ] [sort= ]
  table=schema.tablename   Use 'schema.tablename' with 'explain=' option.
  explain=user/password    Connect to ORACLE and issue EXPLAIN PLAN.
  print=integer    List only the first 'integer' SQL statements.
  pdbtrace=user/password   Connect to ORACLE to retrieve SQL trace records.
  aggregate=yes|no
  .....
  ......
```

### tkprof 실행예
sys=no 옵션은 SQL 파싱하는 과정에서 내부적으로 수행되는 SQL 문을 제외한다.
```
$ tkprof /oracle/u01/app/oracle/diag/rdbms/devdb/devdb/trace/devdb_ora_6096.trc report.prf sys=no
```

```
$ vi report.prf
TKPROF: Release 19.0.0.0.0 - Development on Thu Jun 25 05:37:55 2026

Copyright (c) 1982, 2019, Oracle and/or its affiliates.  All rights reserved.

Trace file: /oracle/u01/app/oracle/diag/rdbms/devdb/devdb/trace/devdb_ora_6096.trc
Sort options: default

********************************************************************************
count    = number of times OCI procedure was executed
cpu      = cpu time in seconds executing
elapsed  = elapsed time in seconds executing
disk     = number of physical reads of buffers from disk
query    = number of buffers gotten for consistent read
current  = number of buffers gotten in current mode (usually for update)
rows     = number of rows processed by the fetch or execute call
********************************************************************************

The following statement encountered a error during parse:

select * from emp wherfe empno = 7900

Error encountered: ORA-00933
********************************************************************************

SQL ID: 8am3c6wqp3upw Plan Hash: 2949544139

select *
from
 emp where empno = 7900


call     count       cpu    elapsed       disk      query    current        rows
------- ------  -------- ---------- ---------- ---------- ----------  ----------
Parse        1      0.00       0.00          0          0          0           0
Execute      1      0.00       0.00          0          0          0           0
Fetch        2      0.00       0.00          0          2          0           1
------- ------  -------- ---------- ---------- ---------- ----------  ----------
total        4      0.00       0.00          0          2          0           1

Misses in library cache during parse: 1
Optimizer mode: ALL_ROWS
Parsing user id: 109
Number of plan statistics captured: 1

Rows (1st) Rows (avg) Rows (max)  Row Source Operation
---------- ---------- ----------  ---------------------------------------------------
         1          1          1  TABLE ACCESS BY INDEX ROWID EMP (cr=2 pr=0 pw=0 time=57 us starts=1 cost=1 size=87 card=1)
         1          1          1   INDEX UNIQUE SCAN PK_EMP (cr=1 pr=0 pw=0 time=30 us starts=1 cost=1 size=0 card=1)(object id 73730)

********************************************************************************

SQL ID: a5ks9fhw2v9s1 Plan Hash: 272002086

select *
from
 dual


call     count       cpu    elapsed       disk      query    current        rows
------- ------  -------- ---------- ---------- ---------- ----------  ----------
Parse        1      0.00       0.00          0          0          0           0
Execute      1      0.00       0.00          0          0          0           0
Fetch        2      0.00       0.00          0          2          0           1
------- ------  -------- ---------- ---------- ---------- ----------  ----------
total        4      0.00       0.00          0          2          0           1

Misses in library cache during parse: 1
```

## 7. 트레이스 결과분석
### call 통계
|**항목**|**설명**|
|---|---|
|**call**|커서의 진행 상태에 따라 Parse, Execute, Fetch 세 개의 Call로 나누어 각각에 대한 통계정보를 보여준다.<br><br>  <br>  <br><br>• **Parse** : SQL을 파싱하고 실행계획을 생성하는 단계<br><br>  <br><br>• **Execute** : SQL 커서를 실행하는 단계<br><br>  <br><br>• **Fetch** : 레코드를 실제로 Fetch하는 단계|
|**count**|Parse, Execute, Fetch 각 단계가 수행된 횟수|
|**cpu**|현재 커서가 각 단계에서 사용한 cpu time|
|**elapsed**|현재 커서가 각 단계를 수행하는 데 소요된 시간|
|**disk**|디스크에서 읽은 블록 수|
|**query**|Consistent 모드로 읽은 블록 수(6장 1절 1항 그림 6-4와 함께 설명한 'MVCC 모델' 참고)|
|**current**|Current 모드로 읽은 블록 수(6장 1절 1항 그림 6-4와 함께 설명한 'MVCC 모델' 참고)|
|**rows**|각 단계에서 읽거나 갱신한 건수|

### I/O 분석
|**SQL 트레이스**|**AutoTrace**|**설명**|
|---|---|---|
|**current**|db block gets|Current 모드로 읽은 블록 수|
|**query**|consistent gets|Consistent 모드로 읽은 블록 수|
|**disk**|physical reads|디스크에서 읽은 블록 수|
|**fetch count**|SQL*Net roundtrips<br><br>  <br><br>to/from client|조회 결과를 전송을 위해 클라이언트가 발행한<br><br>  <br><br>Fetch Call 횟수|
|**fetch rows**|rows processed|조회 건수|

### 실행계획통계
```
Rows (1st) Rows (avg) Rows (max)  Row Source Operation
---------- ---------- ----------  ---------------------------------------------------
         1          1          1  TABLE ACCESS BY INDEX ROWID EMP (cr=2 pr=0 pw=0 time=57 us starts=1 cost=1 size=87 card=1)
         1          1          1   INDEX UNIQUE SCAN PK_EMP (cr=1 pr=0 pw=0 time=30 us starts=1 cost=1 size=0 card=1)(object id 73730)

*

```
* 왼쪽 row 수 : 각 수행단계에서 출력된 로우 수
* 부모는 자식 노드의 값을 포함함 :  테이블 엑세스 단계 EMP cr=2 는  자식노드 PK_EMP인덱스 엑세스 단계 cr=1 을 포함함

## 8. 연관 개념
- [[2026-06-29_(Oracle-SQL분석도구-AUTOTRACE-가이드)]]
- [[2026-06-29_(Oracle-SQL분석도구-EXPLAIN-PLAN-실행계획-가이드)]]
- [[2026-07-01_(Oracle-SQL분석도구-DBMS_XPLAN-심화-가이드)]]
- [[2026-06-15-OT03_(Oracle-실행계획-플랜-해석-방법)]]
