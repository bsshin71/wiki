# MySQL Replication 가이드
- **카테고리**: #DBMS #MySQL
- **태그**: #MySQL #replication #GTID #slave #replica #semi-sync #HA #MHA
- **작성일**: 2026-06-12
- **참조 원본**: [[2026-06-12-01. binary log file position based replication - BigDataTeam]], [[2026-06-12-02. replication with global transaction identifiers - BigDataTeam]], [[2026-06-12-03. semi sync replication - BigDataTeam]], [[2026-06-12-05. replcation command 정리 - BigDataTeam]], [[2026-06-12-07. replication 상황별 조치 - BigDataTeam]], [[2026-06-12-semi replication - BigDataTeam]], [[2026-06-12-replication 재설정 방법 - BigDataTeam]], [[2026-06-12-slave 추가방법(cone statement) - BigDataTeam]], [[2026-06-12-slave 추가방법(gtid based) - BigDataTeam]], [[2026-06-12-slave 추가방법(mysqldump) - BigDataTeam]], [[2026-06-12-slave 의 gtid_executed 를 master와 일치시키기 - BigDataTeam]], [[2026-06-12-멀티 소스 복제 구성 - BigDataTeam]], [[2026-06-12-BOS DB HA(failover) 테스트 - BigDataTeam]]

## 1. 핵심 요약
- MySQL Replication: **Binary Log Position 기반** (구버전)과 **GTID 기반** (권장) 두 방식. GTID가 장애 복구가 단순.
- GTID: `source_uuid:transaction_id` 형식. 모든 서버에서 unique. 자동 포지셔닝(`MASTER_AUTO_POSITION=1`) 지원.
- Semi-Sync: Slave relay log 기록 완료 후 ACK → 데이터 손실 방지. 단, Multi-Source Replication과 동시 사용 불가.
- **MySQL 8.4에서 명령어 변경**: `SLAVE` → `REPLICA`, `MASTER` → `SOURCE`, `CHANGE MASTER TO` → `CHANGE REPLICATION SOURCE TO`

## 2. 상세 설명

### Replication 구조

```
Master (Binary Log 생성: DML, DDL 모든 변경)
  └─→ Slave I/O Thread → Relay Log (로컬 저장)
                ↓
        Slave SQL Thread (Relay Log 실행)

각 Slave는 server-id로 식별. Master는 server-id=1 (unique 필수)
```

---

### Binary Log Position 기반 설정

**Master my.cnf**:
```ini
[mysqld]
log-bin=mysql-bin
server-id=1
innodb_flush_log_at_trx_commit=1   # 내구성
sync_binlog=1                       # 내구성
```

**Replication 계정 생성**:
```sql
CREATE USER 'repl'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;
```

**Master 상태 확인 및 Slave 설정**:
```sql
-- Master에서 Binary Log 위치 확인
SHOW MASTER STATUS;
-- File: mysql-bin.000001, Position: 154

-- Slave에서 설정
CHANGE MASTER TO
  MASTER_HOST='192.168.56.3',
  MASTER_USER='repl',
  MASTER_PASSWORD='password',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=154;
START SLAVE;
SHOW SLAVE STATUS \G;
-- Slave_IO_Running: Yes, Slave_SQL_Running: Yes 확인
```

**mysqldump로 초기 데이터 적재 후 Slave 설정**:
```bash
# Master에서 덤프 (binlog position 포함)
mysqldump --all-databases --master-data=2 --single-transaction \
  --user=root --password=pwd > alldump.sql
# 파일에서 CHANGE MASTER TO 위치 확인
grep CHANGE alldump.sql

# Slave에서 import 후 replication 시작
mysql -uroot -p < alldump.sql
```

---

### GTID 기반 설정 (권장)

**GTID 개요**:
- 형식: `source_uuid:transaction_id` (예: `3E11FA47-71CA-11E1-9E33-C80AA9429562:1-5`)
- 모든 서버에서 unique. 파일/포지션 없이 자동 동기화.
- 시스템 테이블 `mysql.gtid_executed`에 저장.

**Master/Slave my.cnf**:

| 항목 | Master | Slave |
|------|--------|-------|
| `log-bin` | 필수 | 권장 (chain replication 시) |
| `gtid_mode` | ON | ON |
| `enforce_gtid_consistency` | ON | ON |
| `server-id` | unique | unique |
| `log_slave_updates` | - | ON (chain replication 시) |

```ini
[mysqld]
log-bin=mysql-bin
gtid_mode=ON
enforce_gtid_consistency=ON
server-id=1
binlog_format=row
log_slave_updates=ON
```

**Slave에서 GTID 기반 연결**:
```sql
CHANGE MASTER TO
  MASTER_HOST='master_ip',
  MASTER_USER='repl',
  MASTER_PASSWORD='password',
  MASTER_AUTO_POSITION=1;    -- GTID 자동 포지셔닝
START SLAVE;
SHOW SLAVE STATUS \G;
```

---

### Slave 추가 방법 3가지

#### 방법 1: XtraBackup (GTID 기반, 권장)

```bash
# STEP 1: Master에서 압축 없이 백업 수행
xtrabackup --backup --user=bkpuser --password=*** --target-dir=/backup/fullbackup/$(date +%Y-%m-%d)

# STEP 2: GTID 정보 확인
cat /backup/fullbackup/2021-03-04/xtrabackup_binlog_info
# mysql-bin.000076  196  4074d152-...:1-1940334

# STEP 3: Prepare
xtrabackup --prepare --target-dir=/backup/fullbackup/2021-03-04

# STEP 4: Slave 서버에 복사 후 copy-back
systemctl stop mysqld
mv /home/bos/mysql/data /home/bos/mysql/data.bak
mkdir /home/bos/mysql/data
xtrabackup --copy-back --target-dir=/backup/fullbackup/2021-03-04
chown -R bos:mysql /home/bos/mysql/data

# STEP 5: Slave 시작 및 Replication 연결
mysqld --defaults-file=/home/bos/mysql/etc/my.cnf &
mysql> RESET MASTER;
mysql> SET GLOBAL gtid_purged="4074d152-...:1-1940334";
mysql> CHANGE MASTER TO
         MASTER_HOST='master_ip',
         MASTER_USER='repl',
         MASTER_PASSWORD='***',
         MASTER_AUTO_POSITION=1;
mysql> START SLAVE;
mysql> SHOW SLAVE STATUS \G;   -- Slave_IO_Running: Yes, Slave_SQL_Running: Yes
```

#### 방법 2: mysqldump

```sql
-- 1. 복제 계정 생성
CREATE USER `repl`@`%` IDENTIFIED BY 'repl_user_password';
GRANT REPLICATION SLAVE ON *.* TO `repl`@`%`;

-- 2. Source에서 dump
# mysqldump --single-transaction --master-data=2 --set-gtid-purged=ON \
#   --all-databases -uroot -p > source_data.sql

-- 3. Replica에서 import
mysql> RESET MASTER;
mysql> SOURCE source_data.sql;
-- dump 파일 안에 SET GLOBAL gtid_purged="..."가 포함됨

-- 4. Replication 시작
mysql> CHANGE MASTER TO MASTER_HOST='master_ip', MASTER_USER='repl',
       MASTER_PASSWORD='password', MASTER_AUTO_POSITION=1;
mysql> START SLAVE;
mysql> SHOW SLAVE STATUS \G;
```

#### 방법 3: Clone Plugin (MySQL 8.0+)

```sql
-- Donor 설정
INSTALL PLUGIN CLONE SONAME "mysql_clone.so";
CREATE USER clone_user IDENTIFIED BY "clone_password";
GRANT BACKUP_ADMIN ON *.* TO clone_user;
GRANT SELECT ON performance_schema.* TO clone_user;

-- Recipient 설정
INSTALL PLUGIN CLONE SONAME "mysql_clone.so";
SET GLOBAL clone_valid_donor_list = "donor.host.com:3306";
CREATE USER clone_user IDENTIFIED BY "clone_password";
GRANT CLONE_ADMIN ON *.* TO clone_user;

-- Clone 실행
CLONE INSTANCE FROM clone_user@donor.host.com:3306 IDENTIFIED BY "clone_password";

-- Clone 진행 상황 확인
SELECT STATE, CAST(BEGIN_TIME AS DATETIME) AS "START TIME",
       CASE WHEN END_TIME IS NULL THEN
         LPAD(sys.format_time(POWER(10,12)*(UNIX_TIMESTAMP(now())-UNIX_TIMESTAMP(BEGIN_TIME))),10,' ')
       ELSE LPAD(sys.format_time(POWER(10,12)*(UNIX_TIMESTAMP(END_TIME)-UNIX_TIMESTAMP(BEGIN_TIME))),10,' ')
       END AS DURATION
FROM performance_schema.clone_status;

-- Clone 완료 후 Replication 연결
CHANGE MASTER TO MASTER_HOST='master_ip', MASTER_USER='repl',
  MASTER_PASSWORD='repl', MASTER_AUTO_POSITION=1;
START SLAVE;
```

> ⚠️ Clone 후 Slave에 auto.cnf 파일이 Donor와 같을 경우 UUID 충돌 오류 발생:
> `Fatal error: master and slave have equal MySQL server UUIDs`
> → Slave의 auto.cnf 삭제 후 재시작

---

### GTID Slave 재설정 (전체 초기화)

```sql
-- 1. Master 리셋 및 락 잡기
RESET MASTER;
FLUSH TABLES WITH READ LOCK;

-- 2. Master 상태 기록
SHOW MASTER STATUS;   -- 결과 기록

-- 3. 전체 dump
# mysqldump -uroot -p --all-databases --set-gtid-purged=ON > all.sql

-- 4. 락 해제
UNLOCK TABLES;

-- 5. Slave에서
STOP SLAVE;
RESET MASTER;
SHOW GLOBAL VARIABLES LIKE 'gtid_executed';   -- 비어있어야 함

-- 6. 데이터 로드
# mysql -uroot -p < all.sql

-- 7. Replication 재설정
CHANGE MASTER TO MASTER_HOST='master_ip', MASTER_USER='repl',
  MASTER_PASSWORD='password', MASTER_PORT=3306, MASTER_AUTO_POSITION=1;
START SLAVE;
```

---

### GTID slave의 gtid_executed 수동 일치

```sql
-- 1. Master와 Slave의 GTID 확인
-- Master
SHOW MASTER STATUS \G;  -- Executed_Gtid_Set 확인
-- Slave
SHOW GLOBAL VARIABLES LIKE 'gtid_executed';

-- 2. Slave에서 GTID 수동 설정
STOP SLAVE;
RESET MASTER;           -- ⚠️ binary log 삭제 주의
SET GLOBAL gtid_purged = 'MASTER의_GTID_EXECUTED_값';

-- 3. Replication 재연결
CHANGE MASTER TO MASTER_HOST='master_ip', MASTER_USER='repl',
  MASTER_PASSWORD='password', MASTER_AUTO_POSITION=1;
START SLAVE;
SHOW SLAVE STATUS \G;
```

---

### 상황별 조치: GTID Transaction Skip

```sql
-- 오류 발생 TX 건너뛰기
-- show slave status에서 Last_Error와 Executed_Gtid_Set 확인

STOP SLAVE;
SET gtid_next='4074d152-0c4c-11eb-aa27-02f7979ac73c:3';  -- Executed_Gtid_Set 마지막+1
BEGIN;
COMMIT;                        -- 빈 트랜잭션으로 해당 GTID 소비
SET gtid_next='AUTOMATIC';
START SLAVE;
SHOW SLAVE STATUS \G;          -- Last_Errno=0 확인

-- 여러 TX skip (sql_slave_skip_counter - GTID 모드에서는 사용 불가)
-- GTID 모드에서는 위 방법 사용
```

---

### 주요 Replication 명령어

**Master 명령어**:
```sql
SHOW MASTER STATUS;                    -- Binary Log 파일명·포지션·GTID 확인
SHOW SLAVE HOSTS \G;                   -- 연결된 Slave 목록
FLUSH TABLES WITH READ LOCK;           -- 데이터 일관성 락
RESET MASTER;                          -- Binary Log 초기화 (⚠️ 주의)
PURGE BINARY LOGS BEFORE '날짜';       -- 오래된 binlog 삭제
```

**Slave 명령어**:
```sql
START SLAVE;                           -- 복제 시작 (8.4: START REPLICA)
STOP SLAVE;                            -- 복제 중지
RESET SLAVE ALL;                       -- Master 채널 정보까지 삭제
SHOW SLAVE STATUS \G;                  -- 복제 상태 확인
SHOW VARIABLES LIKE 'read_only';       -- Master/Slave 구분
```

---

### Semi-Synchronous Replication

**개요**: Master가 최소 1개 Slave의 ACK(relay log 기록 완료)를 받은 후 commit 완료. 데이터 손실 방지.

```
1. application → commit
2. binary log에 기록
3. storage에 기록
4. slave relay log에 기록
5. slave → master ACK
6. master → application OK
7. slave storage 비동기 적용
```

**설치 및 설정**:
```sql
-- Master
INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
SET GLOBAL rpl_semi_sync_master_enabled=1;
SET GLOBAL rpl_semi_sync_master_timeout=1000;  -- 1초 후 async로 전환

-- Slave
INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';
SET GLOBAL rpl_semi_sync_slave_enabled=1;
-- io_thread 재시작 필요
STOP SLAVE IO_THREAD;
START SLAVE IO_THREAD;
```

**my.cnf 설정 (재시작 후 유지)**:
```ini
# Master
rpl_semi_sync_master_enabled=1
rpl_semi_sync_master_timeout=1000

# Slave
rpl_semi_sync_slave_enabled=1
```

**상태 확인**:
```sql
-- Master에서
SHOW GLOBAL STATUS LIKE 'rpl_semi%';
-- Rpl_semi_sync_master_status: ON (ON이어야 함)
-- Rpl_semi_sync_master_clients: 1 이상 (연결된 semi-sync slave 수)

-- Slave에서
SHOW GLOBAL STATUS LIKE '%semi_sync_slave%';
-- Rpl_semi_sync_slave_status: ON
```

**Semi-Sync 주요 변수**:

| 변수 | 설명 |
|------|------|
| `rpl_semi_sync_master_timeout` | Slave ACK 대기 시간(ms). 초과 시 async 전환. 기본 10000(10초) |
| `rpl_semi_sync_master_wait_for_slave_count` | ACK 받아야 할 Slave 수. 기본 1 |
| `rpl_semi_sync_master_wait_no_slave` | Slave 수 부족 시 동작. ON=semi-sync 유지, OFF=async 전환 |
| `rpl_semi_sync_master_wait_point` | ACK 대기 시점. AFTER_SYNC(기본) 또는 AFTER_COMMIT |

> ⚠️ **Multi-Source Replication과 Semi-Sync 동시 사용 불가** (버그: MySQL Bug#102994)
> → Multi-Source Replication 환경에서는 Semi-Sync 사용하면 `Relay log write failure` 오류 발생

---

### Multi-Source Replication

여러 Master로부터 하나의 Slave로 복제 (채널별로 관리):

```sql
-- stdb1 채널
CHANGE MASTER TO MASTER_HOST="10.11.14.141", MASTER_USER="repl",
  MASTER_PASSWORD="xxxx", MASTER_AUTO_POSITION=1
  FOR CHANNEL 'stdb1';

-- stdb2 채널
CHANGE MASTER TO MASTER_HOST="10.11.14.142", MASTER_USER="repl",
  MASTER_PASSWORD="xxxx", MASTER_AUTO_POSITION=1
  FOR CHANNEL 'stdb2';

-- 채널별 시작/확인
START SLAVE FOR CHANNEL 'stdb1';
START SLAVE FOR CHANNEL 'stdb2';
SHOW SLAVE STATUS \G;   -- Channel_Name 컬럼으로 구분

-- gtid_purged 설정 (두 Master의 GTID 모두 포함)
RESET MASTER;
SET GLOBAL gtid_purged="08b0fb1c-...:1-1628472,2d511b4d-...:1-60231";
```

---

### HA Failover (MHA) 결과 요약

테스트 환경: ProxySQL + MHA + 2대 DB (bosdb1 master, bosdb2 slave)

| 시나리오 | Failover 소요시간 | App 재연결 시간 | 데이터 일치 |
|---------|-----------------|----------------|------------|
| Manual Switch (DB 살아있음) | 9초 | 2초 미만 | ✅ |
| Auto Failover (DB 비정상종료) | 11~12초 | ~15초 | ✅ |
| Failback | 3~4초 | 수초 이내 | ✅ |

**Failover 흐름**: MHA Manager 감지 → Slave → New Master 승격 → ProxySQL Backend 변경 → App 재연결

## 3. 연관 개념 (지식 연결)
- [[2026-06-12_(MySQL-XtraBackup-백업복구-가이드)]] — XtraBackup을 이용한 Slave 추가 백업 준비
- [[2026-06-12_(MySQL-Percona-설치-가이드)]] — my.cnf Replication 설정 전체 예시 포함
