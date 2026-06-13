# MySQL Replication 가이드

- **카테고리**: #DBMS #MySQL #Replication #GTID
- **작성일**: 2026-06-13
- **참조 원본**: 13개 replication 관련 소스 파일

## 1. 핵심 요약

MySQL Replication은 **Binary Log Position 기반**과 **GTID (Global Transaction ID) 기반** 두 가지 방식이 있습니다.
GTID는 트랜잭션을 universally unique하게 식별하므로 장애 복구 및 다중 마스터 구성에서 권장됩니다.
Master와 Slave의 server-id가 unique해야 하며, 복제는 I/O Thread와 SQL Thread의 비동기 실행으로 동작합니다.

---

## 2. Replication의 기본 구조

### 2.1 Master-Slave 복제 흐름

```
Master DB
  ↓
Binary Log (이벤트 기록)
  ↓ (I/O Thread 읽기)
Master로부터 전송
  ↓
Slave Relay Log 저장
  ↓ (SQL Thread 재생)
Slave Local Database 적용
```

### 2.2 주요 특징

- **비동기 복제**: Master commit 후 바로 응답, Slave 적용은 독립적 진행
- **이벤트 기반**: Master의 각 트랜잭션이 Binary Log 이벤트로 기록됨
- **선택적 필터링**: Slave에서 특정 DB/테이블만 적용 가능
- **복구 가능성**: Binary Log 위치 또는 GTID로 특정 지점부터 복제 재개 가능

---

## 3. Binary Log Position 기반 복제

### 3.1 Master 설정

```ini
[mysqld]
# Binary Log 활성화
log-bin=mysql-bin
server-id=1

# 내구성 (성능 하락 가능)
innodb_flush_log_at_trx_commit=1
sync_binlog=1
```

### 3.2 Slave 설정

```ini
[mysqld]
server-id=2
relay-log=relay-bin

# Slave 재시작 후에도 복제 자동 시작
relay-log-recovery=1
```

### 3.3 Slave 복제 시작

```sql
CHANGE MASTER TO
  MASTER_HOST='master_ip',
  MASTER_USER='repl_user',
  MASTER_PASSWORD='password',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=154;

START SLAVE;

-- 복제 상태 확인
SHOW SLAVE STATUS\G
```

---

## 4. GTID 기반 복제 (권장)

### 4.1 GTID 개념

- **Format**: `<source_uuid>:<transaction_id>`
- **Universally Unique**: 복제 토폴로지 전체에서 고유
- **자동 추적**: 트랜잭션별로 자동 식별 및 추적

### 4.2 Master 설정

```ini
[mysqld]
server-id=1
log-bin=mysql-bin

# GTID 활성화 (5.7.6+)
gtid_mode=ON
enforce_gtid_consistency=ON

# Binary Log 변경 시 Slave 자동 재연결
log_slave_updates=ON
```

### 4.3 Slave 설정

```ini
[mysqld]
server-id=2
relay-log=relay-bin

gtid_mode=ON
enforce_gtid_consistency=ON
log_slave_updates=ON

# GTID 자동 위치 결정
skip_slave_start=ON
```

### 4.4 Slave 복제 시작 (GTID)

```sql
-- 자동으로 Master의 모든 GTID를 따라잡음
CHANGE MASTER TO
  MASTER_HOST='master_ip',
  MASTER_USER='repl_user',
  MASTER_PASSWORD='password',
  MASTER_AUTO_POSITION=1;

START SLAVE;

-- 복제 상태 확인
SHOW SLAVE STATUS\G
```

### 4.5 mysql.gtid_executed 테이블

```sql
-- GTID 실행 현황 조회
SELECT * FROM mysql.gtid_executed;

-- GTID Set 조회 (글로벌)
SHOW VARIABLES LIKE 'gtid_executed';

-- 특정 범위까지만 적용
START SLAVE UNTIL SQL_BEFORE_GTIDS='<gtid_set>';
```

---

## 5. Slave 추가 방법 (3가지)

### 5.1 XtraBackup 기반 (권장, 온라인)

```bash
# Master에서 전체 백업
innobackupex --defaults-file=/etc/my.cnf \
  --no-lock --user=root --password=pwd /backup/path

# 백업 준비
innobackupex --apply-log /backup/path

# Slave로 이동 후 복제 시작
# (GTID 자동 감지)
```

### 5.2 Binary Dump + Position

```sql
-- Master에서
SHOW MASTER STATUS;  -- Log File, Position 기록

-- Slave에서
CHANGE MASTER TO
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=154;
START SLAVE;
```

### 5.3 mysqldump (작은 DB용)

```bash
# Master에서
mysqldump --master-data=2 --all-databases > dump.sql

# Slave에서 복원
mysql < dump.sql

# dump.sql의 CHANGE MASTER TO 라인 참고
```

---

## 6. Semi-Synchronous Replication

Slave가 Relay Log를 읽을 때까지 Master가 대기하여 데이터 손실 위험 감소.

### 6.1 Master 설정

```sql
INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
SET GLOBAL rpl_semi_sync_master_enabled=ON;
SET GLOBAL rpl_semi_sync_master_timeout=10000; -- 10초
```

### 6.2 Slave 설정

```sql
INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';
SET GLOBAL rpl_semi_sync_slave_enabled=ON;
```

---

## 7. 주요 Replication 명령어

| 명령어 | 기능 |
|--------|------|
| `SHOW MASTER STATUS;` | Master의 현재 Binary Log 위치 |
| `SHOW SLAVE STATUS\G` | Slave 복제 상태 상세 |
| `FLUSH LOGS;` | 현재 Binary Log 로테이션 |
| `PURGE BINARY LOGS BEFORE ...;` | 오래된 Binary Log 삭제 |
| `SET GLOBAL READ_ONLY=1;` | Slave를 읽기 전용으로 (쓰기 방지) |
| `SHOW VARIABLES LIKE 'gtid%';` | GTID 설정 확인 |

---

## 8. Replication 상황별 조치

### 8.1 Slave가 Master를 따라잡지 못할 때

```sql
-- 1. Slave 상태 확인
SHOW SLAVE STATUS\G
-- Seconds_Behind_Master 값 확인

-- 2. Slave SQL Thread 병목 확인
SELECT * FROM performance_schema.replication_applier_status;

-- 3. 병렬 처리 활성화 (MySQL 5.7+)
SET GLOBAL slave_parallel_workers=4;
SET GLOBAL slave_parallel_type='LOGICAL_CLOCK';

STOP SLAVE;
START SLAVE;
```

### 8.2 GTID 불일치

```sql
-- Slave에서 Master의 특정 GTID 강제 건너뛰기
SET GTID_NEXT='<source_uuid>:<transaction_id>';
BEGIN; COMMIT;
SET GTID_NEXT='AUTOMATIC';
```

### 8.3 Slave 재설정 (전체 초기화)

```sql
STOP SLAVE;
RESET SLAVE ALL;  -- 모든 복제 설정 제거

-- GTID 초기화 (필요 시)
RESET MASTER;

-- 재설정 후 다시 시작
CHANGE MASTER TO ...;
START SLAVE;
```

### 8.4 Replication 상황별 조치 (HA/Failover)

```
1. Master 장애 발생
   ↓
2. Slave 승격 (Slave → New Master)
   - SET GLOBAL READ_ONLY=0;
   - Application 포인트 변경
   
3. Old Master 복구
   ↓
4. Old Master를 New Slave로 설정
   - CHANGE MASTER TO ...
   - START SLAVE;
```

---

## 9. 멀티 소스 복제 (Multi-Source Replication)

하나의 Slave가 여러 Master로부터 복제:

```sql
-- Master 1
CHANGE MASTER TO
  MASTER_HOST='master1_ip',
  MASTER_USER='repl',
  MASTER_PASSWORD='pwd',
  FOR CHANNEL 'master1';

-- Master 2
CHANGE MASTER TO
  MASTER_HOST='master2_ip',
  MASTER_USER='repl',
  MASTER_PASSWORD='pwd',
  FOR CHANNEL 'master2';

START SLAVE FOR CHANNEL 'master1';
START SLAVE FOR CHANNEL 'master2';

-- 상태 확인
SHOW SLAVE STATUS FOR CHANNEL 'master1'\G
```

---

## 10. BOS DB HA(Failover) 테스트 시나리오

### 10.1 정상 구성 확인

- Master: 모든 트랜잭션 기록
- Slave: Master와 동기 상태 확인
- Seconds_Behind_Master = 0

### 10.2 Master 장애 시뮬레이션

```bash
# Master 프로세스 중단
systemctl stop mysql
```

### 10.3 Slave 승격 절차

```sql
-- Slave에서 실행
STOP SLAVE;

-- 남은 Relay Log 재생 대기
SELECT MASTER_LOG_FILE, MASTER_LOG_POS FROM INFORMATION_SCHEMA.PROCESSLIST;

-- 읽기 전용 해제
SET GLOBAL READ_ONLY=0;

-- New Master 역할 시작
RESET SLAVE ALL;

-- Application 포인트 변경
-- (Connection String: 192.168.x.y:3306 → New Slave IP)
```

### 10.4 Old Master 복구 후 Slave로 설정

```sql
-- Old Master 부팅
-- XtraBackup 백업 또는 Binary Log 기반 동기화

CHANGE MASTER TO
  MASTER_HOST='new_master_ip',
  MASTER_USER='repl',
  MASTER_PASSWORD='pwd',
  MASTER_AUTO_POSITION=1;

START SLAVE;
```

---

## 11. 연관 개념

- [[2026-06-13_(MySQL-Lock-Deadlock-모니터링)]]
- [[2026-06-13_(MySQL-XtraBackup-백업복구-가이드)]]
- [[2026-06-12_(MySQL-관리자-쿼리-모음)]]
