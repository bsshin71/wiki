# MySQL Replication 가이드
- **카테고리**: #DBMS #MySQL
- **태그**: #MySQL #replication #GTID #slave #HA
- **작성일**: 2026-06-12
- **참조 원본**: [[2026-06-12-01. binary log file position based replication - BigDataTeam]], [[2026-06-12-02. replication with global transaction identifiers - BigDataTeam]], [[2026-06-12-03. semi sync replication - BigDataTeam]], [[2026-06-12-05. replcation command 정리 - BigDataTeam]], [[2026-06-12-07. replication 상황별 조치 - BigDataTeam]], [[2026-06-12-semi replication - BigDataTeam]], [[2026-06-12-replication 재설정 방법 - BigDataTeam]], [[2026-06-12-slave 추가방법(cone statement) - BigDataTeam]], [[2026-06-12-slave 추가방법(gtid based) - BigDataTeam]], [[2026-06-12-slave 추가방법(mysqldump) - BigDataTeam]], [[2026-06-12-slave 의 gtid_executed 를 master와 일치시키기 - BigDataTeam]], [[2026-06-12-멀티 소스 복제 구성 - BigDataTeam]], [[2026-06-12-BOS DB HA(failover) 테스트 - BigDataTeam]]

## 1. 핵심 요약
- MySQL Replication은 **Binary Log Position 기반**과 **GTID 기반** 두 가지 방식이 있으며, GTID 기반이 장애 시 복구가 간단해 권장됨.
- Slave 추가 시 XtraBackup 사용이 가장 안전하며, GTID 기반은 `RESET MASTER; SET GLOBAL gtid_purged=...;` 필수.
- Slave 오류 시 GTID skip: `SET gtid_next='UUID:N'; BEGIN; COMMIT; SET gtid_next='AUTOMATIC';`

## 2. 상세 설명

### Replication 구조

```
Master (Binary Log 생성)
  └─→ Slave I/O Thread → Relay Log
                ↓
        Slave SQL Thread (Relay Log 실행)
```

- 각 Slave는 Master binary log 전체를 수신하되, 적용 여부는 Slave가 결정
- `server-id`로 서버 식별, 반드시 Master·Slave 간 unique 필요

---

### Binary Log Position 기반 설정

**Master my.cnf**:
```ini
[mysqld]
log-bin=mysql-bin
server-id=1
innodb_flush_log_at_trx_commit=1
sync_binlog=1
```

**Replication 계정 생성**:
```sql
CREATE USER 'repl'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;
```

**Master 상태 확인**:
```sql
SHOW MASTER STATUS;
-- File: mysql-bin.000006, Position: 154
```

**Slave 설정**:
```sql
CHANGE MASTER TO
  MASTER_HOST='192.168.56.3',
  MASTER_USER='repl',
  MASTER_PASSWORD='password',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=6604;
START SLAVE;
SHOW SLAVE STATUS \G;
-- Slave_IO_Running: Yes / Slave_SQL_Running: Yes 확인
```

---

### GTID 기반 설정 (권장)

**Master/Slave 공통 my.cnf**:
```ini
[mysqld]
gtid_mode=ON
enforce_gtid_consistency=ON
log-bin=mysql-bin
server-id=<unique>
log_slave_updates=ON
binlog_format=row
```

**Slave에서 Master 연결 (GTID)**:
```sql
CHANGE MASTER TO
  MASTER_HOST='master_ip',
  MASTER_USER='repl',
  MASTER_PASSWORD='password',
  MASTER_AUTO_POSITION=1;   -- GTID 기반 자동 포지셔닝
START SLAVE;
```

**GTID 형식**: `source_uuid:transaction_id` (예: `3E11FA47-71CA-11E1-9E33-C80AA9429562:1-5`)

---

### Slave 추가 방법 (GTID 기반, XtraBackup 사용)

```bash
# STEP 1: Master에서 압축 없이 백업
xtrabackup --backup --user=bkpuser --password=*** --target-dir=/backup/fullbackup/$(date +%Y-%m-%d)

# STEP 2: xtrabackup_binlog_info에서 GTID 확인
cat /backup/fullbackup/2021-03-04/xtrabackup_binlog_info
# mysql-bin.000076  196  4074d152-...:1-1940334

# STEP 3: Prepare
xtrabackup --prepare --target-dir=/backup/fullbackup/2021-03-04

# STEP 4: 새 Slave에 복사 후 copy-back
systemctl stop mysqld
xtrabackup --copy-back --target-dir=/backup/fullbackup/2021-03-04
chown -R mysql:mysql /home/bos/mysql/data

# STEP 5: Slave 시작 및 Replication 연결
mysql> RESET MASTER;
mysql> SET GLOBAL gtid_purged="4074d152-...:1-1940334";
mysql> CHANGE MASTER TO
         MASTER_HOST='master_ip',
         MASTER_USER='repl',
         MASTER_PASSWORD='***',
         MASTER_AUTO_POSITION=1;
mysql> START SLAVE;
mysql> SHOW SLAVE STATUS \G;
-- Slave_IO_Running: Yes / Slave_SQL_Running: Yes
```

---

### 주요 Replication 명령어

**Master**:
```sql
SHOW MASTER STATUS;                    -- Binlog 파일명·포지션 확인
SHOW SLAVE HOSTS \G;                   -- 연결된 Slave 목록
FLUSH TABLES WITH READ LOCK;           -- 데이터 일관성을 위한 락
RESET MASTER;                          -- Binary log 초기화 (주의)
```

**Slave**:
```sql
START SLAVE;                           -- 복제 시작
STOP SLAVE;                            -- 복제 중지
RESET SLAVE ALL;                       -- Master 채널 정보까지 삭제
SHOW SLAVE STATUS \G;                  -- 복제 상태 확인
SHOW VARIABLES LIKE 'read_only';       -- Master/Slave 구분
```

---

### 상황별 조치

**GTID Transaction Skip** (오류 발생 TX 건너뛰기):
```sql
-- show slave status에서 Last_Error와 Executed_Gtid_Set 확인
STOP SLAVE;
SET gtid_next='4074d152-...:3';    -- Executed_Gtid_Set의 마지막 번호+1
BEGIN;
COMMIT;
SET gtid_next='AUTOMATIC';
START SLAVE;
SHOW SLAVE STATUS \G;              -- Last_Errno=0 확인
```

**Slave 전체 재설정**:
```sql
STOP SLAVE;
RESET SLAVE ALL;
CHANGE MASTER TO ...;
START SLAVE;
```

---

### Semi-Synchronous Replication

- Master가 최소 1개 Slave의 응답을 받은 후 commit 완료
- 데이터 손실 방지 가능하지만 성능 오버헤드 있음

```sql
-- Master
INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
SET GLOBAL rpl_semi_sync_master_enabled=1;
SET GLOBAL rpl_semi_sync_master_timeout=10000;  -- 10초

-- Slave
INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';
SET GLOBAL rpl_semi_sync_slave_enabled=1;
```

## 3. 연관 개념 (지식 연결)
- [[2026-06-12_(MySQL-XtraBackup-백업복구-가이드)]] — XtraBackup을 이용한 Slave 추가 백업 준비
- [[2026-06-12_(MySQL-Percona-설치-가이드)]] — my.cnf 설정 전체 예시 (Replication 설정 포함)
