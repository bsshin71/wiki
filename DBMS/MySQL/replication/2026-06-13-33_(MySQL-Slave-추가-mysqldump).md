# MySQL Slave 추가 — mysqldump

- **카테고리**: #DBMS #MySQL #Replication
- **태그**: #replication #slave추가 #mysqldump #GTID
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-slave 추가방법(mysqldump) - BigDataTeam]]

## 1. 핵심 요약

논리 백업(mysqldump)으로 신규 Slave를 구성하는 방법입니다.
`--master-data=2 --single-transaction`으로 일관된 덤프를 받고, 덤프에 포함된 `SET GLOBAL gtid_purged`로 GTID를 자동 설정한 뒤 `MASTER_AUTO_POSITION=1`로 복제를 시작합니다. 데이터량이 적을 때 적합합니다.

---

## 2. 복제 계정 준비 (Master)

```sql
CREATE USER `repl`@`%` IDENTIFIED BY 'repl_user_password';
GRANT REPLICATION SLAVE ON *.* TO `repl`@`%`;
```

## 3. mysqldump (Source DB)

```bash
mysqldump --single-transaction --master-data=2 --opt \
  --routines --triggers --hex-blob \
  -uroot -p**** --all-databases > source_data.sql
```

## 4. data import (Replica DB)

```sql
RESET MASTER;
SOURCE source_data.sql;
-- 덤프 파일 안에 SET GLOBAL gtid_purged = "XXXX" 가 포함되어 있음
```

## 5. 복제 생성 및 시작 (Replica DB)

```sql
CHANGE MASTER TO
  MASTER_HOST="$masterip", MASTER_USER="repl",
  MASTER_PASSWORD="$slavepass", MASTER_AUTO_POSITION=1;
START SLAVE;
```

## 6. 복제 확인

```sql
SHOW SLAVE STATUS \G
-- Slave_IO_State: Waiting for source to send event
-- Master_Log_File: binlog.000002
-- Read_Master_Log_Pos: 89057451
```

---

## 7. 연관 개념

- [[2026-06-13-31_(MySQL-Slave-추가-CLONE-플러그인)]]
- [[2026-06-13-32_(MySQL-Slave-추가-XtraBackup-GTID)]]
- [[2026-06-13-34_(MySQL-Replication-재설정)]]
