# MySQL LOAD DATA LOCAL INFILE — 파일 임포트

- **카테고리**: #DBMS #MySQL #tools
- **태그**: #MySQL #tools #load-data-infile #import #local_infile
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-load into infile 사용법 - BigDataTeam]]

## 1. 핵심 요약

`LOAD DATA LOCAL INFILE`로 **로컬 파일을 테이블로 고속 임포트**. 서버(`local_infile=ON`)와 클라이언트(`loose-local-infile=1`) 양쪽 모두 활성화가 필요하다.

---

## 2. 서버 설정

```ini
# my.cnf (재시작 필요)
[mysqld]
local_infile=1
```
```sql
-- 재시작 불가 시 런타임 변경
SET GLOBAL local_infile = 1;
SHOW GLOBAL VARIABLES LIKE 'local_infile';
```

## 3. 클라이언트 설정

```ini
# ~/.my.cnf
[client]
loose-local-infile=1
```

## 4. LOAD 명령

```sql
LOAD DATA LOCAL INFILE '/path/to/file/data.tsv'
INTO TABLE my_table_name
FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
(`col1`, `col2`, `col3`, `col4`);
```

> `FIELDS TERMINATED BY`로 구분자 지정 (탭: `'\t'`, 콤마: `','`)

## 5. 연관 개념

- [[2026-06-14-MS40_(MySQL-Percona-Toolkit-도구-모음)]] — pt-archiver로 선택적 export 후 LOAD INFILE 임포트
