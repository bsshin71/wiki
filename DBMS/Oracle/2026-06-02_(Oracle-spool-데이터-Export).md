# Oracle spool 데이터 Export
- **카테고리**: #DBMS #Oracle
- **태그**: #Oracle #export #spool #유틸리티
- **작성일**: 2026-06-02
- **참조 원본**: raw/2026-06-02_spool로 export하는쿼리.md

## 1. 핵심 요약
- Oracle SQL*Plus의 spool 명령을 활용하여 테이블 데이터를 SQL INSERT 형태로 export하는 스크립트. 구분자 기반 CSV-like 형식으로 출력.

## 2. 상세 설명

### getdata.sql — 데이터 Export 스크립트

```sql
SET SERVEROUTPUT on;
SET TIMING off;
SET FEEDBACK off;
SET VERIFY off;
SET LINESIZE 2000;
SET TRIMSPOOL ON;

define TABLE_NAME=&&1

spool &&TABLE_NAME..sql
EXEC DBMS_OUTPUT.ENABLE(5000);
EXEC DBMS_OUTPUT.PUT_LINE('SET NEWPAGE 0;');
EXEC DBMS_OUTPUT.PUT_LINE('SET LINESIZE 2000;');
EXEC DBMS_OUTPUT.PUT_LINE('SET PAGESIZE 0;');
EXEC DBMS_OUTPUT.PUT_LINE('SET HEADING OFF;');
EXEC DBMS_OUTPUT.PUT_LINE('SET TRIMSPOOL ON;');
EXEC DBMS_OUTPUT.PUT_LINE('ALTER SESSION SET NLS_DATE_FORMAT=''YYYY/MM/DD HH24:MI:SS'';');
EXEC DBMS_OUTPUT.PUT_LINE('spool '||'&&TABLE_NAME..dat');

DECLARE
  column_name   varchar2(40);
  last_column   varchar2(40);
  CURSOR C1(last_column VARCHAR2) IS
    SELECT DECODE(COLUMN_NAME, last_column, COLUMN_NAME||'||''[구분자]''',
                               COLUMN_NAME||'||''^C-c^''||')
    FROM USER_TAB_COLUMNS
    WHERE TABLE_NAME = UPPER('&&TABLE_NAME')
    ORDER BY COLUMN_ID;
BEGIN
  SELECT COLUMN_NAME INTO last_column
  FROM USER_TAB_COLUMNS
  WHERE TABLE_NAME = UPPER('&&TABLE_NAME')
    AND COLUMN_ID = (SELECT MAX(COLUMN_ID) FROM USER_TAB_COLUMNS WHERE TABLE_NAME = UPPER('&&TABLE_NAME'));

  DBMS_OUTPUT.PUT_LINE('SELECT');
  OPEN C1(last_column);
  LOOP
    FETCH C1 INTO column_name;
    EXIT WHEN C1%NOTFOUND;
    DBMS_OUTPUT.PUT_LINE(column_name);
  END LOOP;
  CLOSE C1;
  DBMS_OUTPUT.PUT_LINE('FROM &&TABLE_NAME;');
END;
/
EXEC DBMS_OUTPUT.PUT_LINE('spool off;');
EXEC DBMS_OUTPUT.PUT_LINE('exit;');
spool off;
start &&TABLE_NAME..sql
```

### 실행 절차

```bash
# 1. 테이블 생성 스크립트 실행
sqlplus scott/tiger < crt.sql

# 2. 데이터 INSERT 스크립트 실행
sqlplus scott/tiger < ins.sql

# 3. spool export 실행
sh -v run.sh    # → .dat 파일 생성

# 4. 형식 변환
sh -v formout.sh

# 5. Import 실행
sh -v in.sh
```

### run.sh 예시

```bash
sqlplus -S mdbbmt/mdbbmt @getdata ETL1
```

## 3. 연관 개념 (지식 연결)
- [[2026-06-02_(Oracle-모니터링-관리-쿼리-모음)]] — Oracle 관련 쿼리 모음
