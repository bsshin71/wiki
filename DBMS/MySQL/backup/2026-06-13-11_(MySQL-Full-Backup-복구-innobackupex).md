# MySQL Full Backup & 복구 (innobackupex)

- **카테고리**: #DBMS #MySQL #Backup
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-Full 백업 및 복구 - BigDataTeam.md]]

## 1. 핵심 요약

**innobackupex**(Percona XtraBackup 2.4 이하)는 InnoDB뿐 아니라 MyISAM, Archive 등 모든 스토리지 엔진을 **논리적 락 없이** 백업합니다.
백업 중 변경된 트랜잭션은 LSN(Log Sequence Number) 기반으로 xtrabackup_logfile에 기록되며,
복구 시 **--apply-log**로 준비한 후 **--copy-back**으로 원본 위치에 복원합니다.

---

## 2. Full Backup 절차

### 2.1 백업 스크립트

```bash
#!/bin/bash
innobackupex \
  --defaults-file=/etc/my.cnf \
  --no-lock \
  --user=backupuser \
  --password=password \
  /backup/path
```

**주요 옵션:**
- `--no-lock`: 테이블 락 없이 백업 (MyISAM은 일관성 주의)
- `--user`: 백업용 MySQL 사용자
- `--password`: 비밀번호
- 마지막 인자: 백업 저장 경로

### 2.2 백업 실행

```bash
$ sh backup.sh > backup.log 2>&1
```

### 2.3 백업 실행 로그 분석

**Step 1: 소프트웨어 정보**
```
Using server version 5.7.23-log
innobackupex version 2.4.15 based on MySQL server 5.7.19
```

**Step 2: LSN 시작점 기록**
```
log scanned up to (805714803)  ← 백업 시작 LSN
```

**Step 3: InnoDB 테이블 백업**
```
Copying ./ibdata1 to /backup/2019-06-30_04-08-09/ibdata1
[01] ...done
```

**Step 4: Non-InnoDB 테이블 백업**
```
Starting to backup non-InnoDB tables and files
Copying ./mysql/slave_master_info.frm to /backup/.../mysql/slave_master_info.frm
```

**Step 5: LSN 종료점 기록**
```
xtrabackup: The latest check point (for incremental): '805714794'
log scanned up to (805714803)
```

**Step 6: 백업 정보 파일 작성**
```
Writing /backup/.../xtrabackup_binlog_info
Writing /backup/.../xtrabackup_info
Transaction log of lsn (805714794) to (805714803) was copied.
completed OK!
```

---

## 3. 백업 결과 파일

### 3.1 생성되는 파일

| 파일명 | 크기 | 용도 |
|--------|------|------|
| `xtrabackup_binlog_info` | 20 bytes | 백업 완료 시점의 Binary Log 파일명과 포지션 |
| `xtrabackup_checkpoints` | 150 bytes | 백업 타입(full/incremental), LSN 정보 |
| `xtrabackup_info` | 500+ bytes | innobackupex 옵션, 백업 메타정보 |
| `xtrabackup_logfile` | 수GB | 백업 중 변경된 트랜잭션 로그 (필수) |
| `ibdata1, *.ibd` | 수GB | InnoDB 테이블 데이터 파일 |
| `*.frm, *.MYI, *.MYD` | 수MB | 테이블 정의 및 MyISAM 데이터 |

### 3.2 메타데이터 예시

**xtrabackup_binlog_info:**
```
bin_log.000002 9257
```

**xtrabackup_checkpoints:**
```
backup_type = full-backuped
from_lsn = 0
to_lsn = 805714794
last_lsn = 805714803
flushed_lsn = 805714803
```

**xtrabackup_info:**
```
uuid = 3e833212-9aa1-11e9-a7a8-080027ef2499
tool_name = innobackupex
tool_version = 2.4.15
server_version = 5.7.23-log
start_time = 2019-06-30 04:08:10
end_time = 2019-06-30 04:08:14
binlog_pos = filename 'bin_log.000002', position '9257'
incremental = N
```

---

## 4. 복구 절차

### 4.1 복구 준비 (--apply-log)

```bash
innobackupex \
  --defaults-file=/etc/my.cnf \
  --apply-log \
  /backup/2019-06-30_04-08-09
```

**동작:**
1. xtrabackup_logfile의 트랜잭션 로그 재생
2. 마지막 LSN 포인트로 일관성 복구
3. 복구용 InnoDB Redo Log 생성

**로그 분석:**
```
xtrabackup: innodb_data_home_dir = .
xtrabackup: innodb_data_file_path = ibdata1:12M:autoextend

xtrabackup: Starting InnoDB instance for recovery

InnoDB: Last MySQL binlog file position 8566, file name bin_log.000002

InnoDB: Setting log file ./ib_logfile0 size to 48 MB
InnoDB: New log files created, LSN=805715234

InnoDB: Starting crash recovery
InnoDB: Shutdown completed; log sequence number 805715496

completed OK!
```

### 4.2 MySQL 종료 및 기존 데이터 백업

```bash
# MySQL 종료
systemctl stop mysqld.service

# 기존 데이터 백업
cp -r /var/lib/mysql /backup/old_mysql_backup

# 데이터 디렉터리 초기화
rm -rf /var/lib/mysql
mkdir -p /var/lib/mysql
```

### 4.3 백업 복원 (--copy-back)

```bash
innobackupex \
  --defaults-file=/etc/my.cnf \
  --copy-back \
  /backup/2019-06-30_04-08-09

# MySQL 계정 권한 설정
chown -R mysql:mysql /var/lib/mysql
```

**주요 검사:**
- 모든 데이터 파일이 올바르게 복사됨
- 권한이 mysql:mysql으로 설정됨

### 4.4 MySQL 시작 및 검증

```bash
# MySQL 시작
systemctl start mysqld.service

# 접속 확인
mysql -uroot -p -e "SELECT VERSION(); SHOW TABLES FROM mysql;"
```

---

## 5. 에러 처리

### 5.1 파일 권한 오류

**증상:**
```
[ERROR] InnoDB: os_file_get_status() failed on './ibdata1'
Can't determine file permissions
```

**원인:** SELinux 정책 또는 파일 소유권 오류

**해결:**
```bash
# SELinux 비활성화
setenforce 0

# 또는 권한 확인
chown -R mysql:mysql /var/lib/mysql
chmod 750 /var/lib/mysql
```

### 5.2 Socket 연결 오류

**증상:**
```
ERROR 2002 (HY000): Can't connect to local MySQL server 
through socket '/var/lib/mysql/mysql.sock' (2)
```

**원인:** 데이터 디렉터리 권한 부족

**해결:**
```bash
systemctl stop mysqld.service
chmod 777 /var/lib/mysql
systemctl start mysqld.service
```

### 5.3 LSN 불일치 오류

**증상:**
```
[ERROR] InnoDB: Redo log is not ready for crash recovery
```

**원인:** 백업 후 추가 트랜잭션 발생

**해결:**
- 백업 파일을 다시 --apply-log로 준비
- 또는 Fresh Backup 수행

---

## 6. Incremental Backup과의 차이

| 항목 | Full Backup | Incremental Backup |
|------|------------|-------------------|
| 백업 크기 | 전체 데이터 | 마지막 LSN 이후 변경분만 |
| 복구 시간 | 빠름 | 느림 (Full + Incremental 모두 적용) |
| 저장소 효율 | 낮음 | 높음 |
| 복구 복잡도 | 간단 | 복잡 |
| 사용 시기 | 일주일 1회 | 매일 |

---

## 7. 체크리스트

| 항목 | 확인 |
|------|------|
| **백업 스크립트** | ✅ 사용자/비밀번호 정확 |
| **저장 경로** | ✅ 충분한 용량 (데이터 크기의 110%) |
| **--apply-log** | ✅ 복구 전 필수 |
| **파일 권한** | ✅ mysql:mysql 소유권 |
| **SELinux** | ✅ 비활성화 또는 정책 설정 |
| **Binary Log** | ✅ binlog_pos 정확 |

---

## 8. 연관 개념

- [[2026-06-13-01_(MySQL-Binary-Log-Position-복제)]]
- [[2026-06-13-04_(MySQL-복제-명령어-모음)]]
