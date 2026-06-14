# PostgreSQL mysql_fdw (MySQL 연동)

- **카테고리**: #DBMS #PostgreSQL #연동
- **태그**: #PostgreSQL #FDW #mysql_fdw #foreign_table #migration
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-FDW 를 활용한 mysql 과의 연동 - BigDataTeam]]

## 1. 핵심 요약

`mysql_fdw` 확장으로 PostgreSQL에서 **MySQL 테이블을 외부 테이블(foreign table)로 직접 조회·이관**할 수 있습니다.
서버(SERVER)·사용자 매핑(USER MAPPING)·외부 스키마를 설정하고, `SELECT`/`INSERT INTO ... SELECT`로 데이터를 이관합니다. 필요 시 MySQL 쪽에 view로 변환(예 `0000-00-00` → NULL)합니다.

---

## 2. FDW 설치 (오프라인 패키지)

```bash
mkdir -p ~/pg16_mysql_fdw && cd ~/pg16_mysql_fdw
dnf download --resolve mysql_fdw_16 mariadb-connector-c
# 오프라인 서버로 옮긴 후
dnf install -y *.rpm
```

## 3. 설정 절차

```sql
-- 1) extension
CREATE EXTENSION IF NOT EXISTS mysql_fdw;

-- 2) MySQL 서버 등록
CREATE SERVER mysql_srv_dcrm FOREIGN DATA WRAPPER mysql_fdw
  OPTIONS (host '10.11.4.143', port '3306');

-- 3) 사용자 매핑
CREATE USER MAPPING FOR postgres SERVER mysql_srv_dcrm
  OPTIONS (username 'root', password '****');

-- 4) 외부 테이블용 스키마
CREATE SCHEMA IF NOT EXISTS ext_mysql_dcrm;
IMPORT FOREIGN SCHEMA "CRM" FROM SERVER mysql_srv_dcrm INTO ext_mysql_dcrm;  -- 또는 CREATE FOREIGN TABLE
```

## 4. MySQL 쪽 변환 view (선택)

> MySQL의 `0000-00-00 00:00:00` 등 PG 비호환 값은 view로 정제.

```sql
-- MySQL
CREATE OR REPLACE VIEW v_client AS
SELECT client_id, login_id, email,
       NULLIF(update_time, '0000-00-00 00:00:00') AS update_time
FROM client;
```

## 5. 조회 / 이관

```sql
SELECT * FROM ext_mysql_dcrm.client LIMIT 10;
-- 데이터 이관
INSERT INTO local_client SELECT * FROM ext_mysql_dcrm.client;
```

## 6. 연관 개념

- [[2026-06-14-PG06_(PostgreSQL-스키마-관리-search_path)]]
- [[2026-06-02_(MySQL-데이터이관-Shell-스크립트)]]
