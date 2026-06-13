# MySQL Binary Log Position 기반 복제

- **카테고리**: #DBMS #MySQL #Replication
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-01. binary log file position based replication - BigDataTeam.md]]

## 1. 핵심 요약

Binary Log Position 기반 복제는 Master의 Binary Log 파일과 위치(position)를 기준으로 Slave가 변경사항을 따라잡는 방식입니다.
Master에서 binary logging을 활성화하고, Slave는 CHANGE MASTER TO로 로그 파일과 위치를 지정한 후 START SLAVE로 복제를 시작합니다.
초기 데이터 동기화는 mysqldump 또는 cold backup으로 진행합니다.

---

## 2. 복제 구조 및 동작 원리

### 2.1 Binary Log 기반 복제의 기본 개념

- **Binary Log**: Master의 모든 데이터 변경사항을 event로 기록
  - 1개 event = 1개 transaction (InnoDB 기준)
  
- **Relay Log**: Slave가 Master의 Binary Log를 읽어 로컬에 저장한 로그
  - Slave의 I/O Thread가 관리
  
- **복제 스레드**:
  - I/O Thread: Master의 Binary Log를 읽어 Relay Log에 저장
  - SQL Thread: Relay Log의 event를 Slave 데이터베이스에 재생

### 2.2 주요 특징

- Binary Log 파일과 position으로 복제 위치 추적
- 각 Slave가 독립적으로 Binary Log event 필터링 가능 (특정 DB/테이블 선택)
- 여러 Slave 동시 연결 가능 (각각 다른 처리 가능)
- 장애 시 마지막 처리 위치부터 복제 재개 가능
- Master와 Slave는 unique-id (**server-id**)로 구분 필수

---

## 3. Master 설정

### 3.1 my.cnf 설정

```ini
[mysqld]

# Binary Log 활성화
log-bin=mysql-bin
server-id=1

# 내구성 (성능 하락 가능)
innodb_flush_log_at_trx_commit=1  # 각 commit 시 disk에 flush
sync_binlog=1                      # Binary Log도 disk에 동기화

# 선택사항
# skip-networking                  # Unix socket 접속만 허용
```

### 3.2 설정 확인

```sql
SHOW VARIABLES LIKE 'log_bin%';
SHOW VARIABLES LIKE 'server_id';
SHOW VARIABLES LIKE 'sync_binlog';
SHOW VARIABLES LIKE 'innodb_flush_log_at_trx_commit';
```

---

## 4. Replication 권한 설정

### 4.1 Replication 사용자 생성

```sql
CREATE USER 'repl_user'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'%';
FLUSH PRIVILEGES;
```

### 4.2 권한 확인

```sql
SHOW GRANTS FOR 'repl_user'@'%';
```

---

## 5. Binary Log 파일 및 Position 얻기

### 5.1 방법 1: SHOW MASTER STATUS (실시간)

```sql
SHOW MASTER STATUS;

-- 출력 예:
-- File: mysql-bin.000006
-- Position: 154
```

**사용 시기**: 데이터 준비가 완료되어 즉시 복제를 시작하는 경우

### 5.2 방법 2: mysqldump로 백업 및 Position 추출

```bash
# Master에서 (모든 write 중단 후)
mysqldump --all-databases --opt --triggers --routines \
  --master-data=2 --single-transaction \
  --user=backup_user --password > backup.sql

# backup.sql에서 CHANGE MASTER TO 라인 추출
grep "CHANGE MASTER TO" backup.sql
# 출력: -- CHANGE MASTER TO MASTER_LOG_FILE='mysql-bin.000006', MASTER_LOG_POS=154;
```

---

## 6. 초기 데이터 동기화

### 6.1 mysqldump 방식 (온라인)

#### 단계 1: Master에서 Read Lock 설정

```sql
-- Session 1 (계속 유지)
FLUSH TABLES WITH READ LOCK;
```

#### 단계 2: 다른 세션에서 백업 실행

```bash
# Session 2
mysqldump --all-databases --master-data=2 \
  --user=root --password --host=master_ip \
  --single-transaction --flush-privileges \
  --add-drop-database --add-drop-table \
  --routines --triggers > backup.sql
```

#### 단계 3: Slave에 적용

```bash
mysql --user=root --password --host=slave_ip < backup.sql
```

#### 단계 4: Lock 해제

```sql
-- Session 1에서
UNLOCK TABLES;
```

### 6.2 Cold Backup 방식 (오프라인)

Master가 대용량 데이터인 경우 효율적:

```bash
# Master 종료
mysqladmin -h 127.0.0.1 -uroot -p shutdown

# datadir 압축
tar cvfz mysql_data.tar.gz /path/to/datadir

# Slave로 이동 및 복원
tar xvfz mysql_data.tar.gz

# Slave my.cnf에서 server-id 변경 후 시작
```

---

## 7. Slave 설정

### 7.1 Slave my.cnf 설정

```ini
[mysqld]
server-id=2
# 다른 설정은 Master와 유사하되 server-id만 고유해야 함
```

### 7.2 Slave 기동 및 복제 시작

```bash
# Slave 시작
mysqld_safe --defaults-file=/path/to/my.cnf &
```

```sql
-- Slave에 접속
CHANGE MASTER TO
  MASTER_HOST='master_ip',
  MASTER_USER='repl_user',
  MASTER_PASSWORD='password',
  MASTER_LOG_FILE='mysql-bin.000006',
  MASTER_LOG_POS=154;

START SLAVE;

-- 상태 확인
SHOW SLAVE STATUS\G
```

### 7.3 복제 상태 확인

```
Slave_IO_Running: Yes    ← Master와의 통신 성공
Slave_SQL_Running: Yes   ← Relay Log 재생 중
Seconds_Behind_Master: 0 ← 동기 상태
```

---

## 8. 복제 검증

### 8.1 Master에서 변경 생성

```sql
-- Master
USE test;
CREATE TABLE t1 (id INT);
INSERT INTO t1 VALUES (1);
COMMIT;
```

### 8.2 Slave에서 확인

```sql
-- Slave (자동 적용됨)
USE test;
SHOW TABLES;  -- t1 테이블 존재 확인
SELECT * FROM t1;  -- 데이터도 동기화됨
```

---

## 9. 주요 명령어

| 명령어 | 기능 |
|--------|------|
| `SHOW MASTER STATUS` | Master의 현재 Binary Log 상태 |
| `SHOW SLAVE STATUS` | Slave의 복제 상태 (상세) |
| `START SLAVE` | Slave 복제 시작 |
| `STOP SLAVE` | Slave 복제 중단 |
| `CHANGE MASTER TO` | Slave의 Master 정보 설정 |
| `RESET SLAVE` | Slave 복제 정보 초기화 |
| `FLUSH LOGS` | Binary Log 로테이션 |

---

## 10. 장점 및 단점

### 장점
- 구현이 간단하고 직관적
- 대부분의 MySQL 버전에서 지원
- 초기 setup이 쉬움

### 단점
- Binary Log 파일/position 수동 관리
- Slave 추가 시 position 정확도 필요
- Master 변경 시 복잡한 절차 필요
- 위치 기반이라 정확한 동기화 어려움

---

## 11. 주의사항

1. **server-id 중복 금지**: Master와 Slave의 server-id는 반드시 다를 것
2. **초기 데이터 동기화**: Binary Log Position만으로는 부족, 반드시 데이터 동기화 필요
3. **Binary Log 보관**: Slave가 따라잡을 때까지 Master의 Binary Log 유지 필요
4. **네트워크**: Master-Slave 간 안정적인 네트워크 필수

---

## 12. 연관 개념

- [[2026-06-13-02_(MySQL-GTID-기반-복제)]]
- [[2026-06-13-03_(MySQL-Semi-Sync-복제)]]
- [[2026-06-13-04_(MySQL-복제-명령어-모음)]]
- [[2026-06-12_(MySQL-Replication-가이드)]] (구 통합 문서)
