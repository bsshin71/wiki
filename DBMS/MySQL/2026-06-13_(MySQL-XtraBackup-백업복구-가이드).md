# MySQL XtraBackup 백업복구 가이드

- **카테고리**: #DBMS #MySQL #Backup #XtraBackup
- **작성일**: 2026-06-13
- **참조 원본**: 14개 backup/XtraBackup 관련 소스 파일

## 1. 핵심 요약

XtraBackup은 MySQL InnoDB 데이터베이스의 **온라인 물리 백업** 도구로, 운영 중단 없이 전체/증분 백업이 가능합니다.
주요 특징은 **Lock-Free 백업** (Non-blocking), **빠른 복구**, **증분 백업 지원**입니다.
Full Backup → 증분 Backup → 준비(prepare) → 복원(restore) 절차로 시점 복구(Point-in-Time Recovery)를 구현합니다.

---

## 2. XtraBackup 설치

### 2.1 YUM/RPM 설치 (권장)

```bash
# Percona 리포지토리 추가
yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm

# XtraBackup 설치
yum install percona-xtrabackup-80

# 버전 확인
xtrabackup --version
```

### 2.2 의존성 확인

```bash
# libaio 확인
ldd /usr/bin/xtrabackup | grep libaio

# 필요 시 설치
yum install libaio
```

---

## 3. Full 백업 및 복구

### 3.1 Full Backup 수행

```bash
#!/bin/bash
# Full Backup Script

BACKUP_DIR="/backup/mysql_$(date +%Y%m%d_%H%M%S)"

innobackupex \
  --defaults-file=/etc/my.cnf \
  --no-lock \
  --user=backupuser \
  --password='password' \
  $BACKUP_DIR

echo "Backup completed: $BACKUP_DIR"
```

### 3.2 출력 분석

```
innobackupex version 2.4.15 based on MySQL server 5.7.19

.. log scanned up to (805714803)       ← LSN (Log Sequence Number)
190630 04:08:10 All tables unlocked    ← Lock 해제 완료
190630 04:08:10 Copy-back completed    ← 복사 완료
```

### 3.3 Full Backup 준비 (Prepare)

```bash
# Redo Log 적용
innobackupex --apply-log $BACKUP_DIR

# InnoDB 테이블 준비 완료
```

### 3.4 Full Backup 복원

```bash
# 백업 파일 권한 확인
ls -la $BACKUP_DIR

# Slave로 복원 (기존 데이터 제거)
rm -rf /var/lib/mysql/*
innobackupex --copy-back $BACKUP_DIR

# 권한 복구
chown -R mysql:mysql /var/lib/mysql

# MySQL 서비스 시작
systemctl start mysql
```

---

## 4. 증분 백업 (Incremental Backup)

### 4.1 Full Backup 후 증분 시작

```bash
#!/bin/bash

FULL_BACKUP_DIR="/backup/full_backup"
INCREMENTAL_DIR="/backup/incremental_$(date +%Y%m%d_%H%M%S)"

# Full Backup (처음 한 번)
innobackupex \
  --defaults-file=/etc/my.cnf \
  --no-lock \
  --user=backupuser \
  --password='password' \
  $FULL_BACKUP_DIR

# Incremental Backup (매일)
innobackupex \
  --defaults-file=/etc/my.cnf \
  --incremental \
  --incremental-basedir=$FULL_BACKUP_DIR \
  --user=backupuser \
  --password='password' \
  $INCREMENTAL_DIR
```

### 4.2 증분 백업 준비 (Prepare with Incremental)

```bash
# 1. Full Backup 준비
innobackupex --apply-log --redo-only $FULL_BACKUP_DIR

# 2. Incremental Backup 적용 (Full + Incr1 + Incr2 ...)
innobackupex --apply-log --redo-only \
  --incremental-dir=$INCREMENTAL_DIR1 \
  $FULL_BACKUP_DIR

innobackupex --apply-log --redo-only \
  --incremental-dir=$INCREMENTAL_DIR2 \
  $FULL_BACKUP_DIR

# 3. 마지막 incremental은 --redo-only 없이 최종 적용
innobackupex --apply-log \
  --incremental-dir=$INCREMENTAL_DIRn \
  $FULL_BACKUP_DIR
```

### 4.3 증분 백업 복원

```bash
# Full + Incremental 통합 후 복원
innobackupex --copy-back $FULL_BACKUP_DIR
chown -R mysql:mysql /var/lib/mysql
systemctl start mysql
```

---

## 5. 시점 복구 (Point-in-Time Recovery, PITR)

### 5.1 Binary Log 활성화 확인

```sql
SHOW VARIABLES LIKE 'log_bin%';
-- log_bin: ON
-- binlog_format: ROW (권장) 또는 MIXED
```

### 5.2 PITR 절차

```bash
#!/bin/bash

BACKUP_DIR="/backup/full_backup"
BINARY_LOG_DIR="/var/log/mysql"
RECOVERY_TIME="2026-06-13 14:00:00"  # 복구 목표 시점

# 1. Full Backup 준비
innobackupex --apply-log $BACKUP_DIR

# 2. Backup 내 LSN 확인
LSN=$(grep "binlog offset" $BACKUP_DIR/xtrabackup_info | awk '{print $NF}')
echo "Backup LSN: $LSN"

# 3. Backup 이후 Binary Log 추출
mysqlbinlog \
  --start-position=$LSN \
  --stop-datetime="$RECOVERY_TIME" \
  $BINARY_LOG_DIR/mysql-bin.* > /tmp/recovery.sql

# 4. 복원
innobackupex --copy-back $BACKUP_DIR
chown -R mysql:mysql /var/lib/mysql

# 5. Binary Log 적용
mysql < /tmp/recovery.sql

# 6. 검증
systemctl start mysql
```

---

## 6. 긴급 장애 복구 (innodb_force_recovery)

InnoDB 손상 시 최소한의 데이터라도 추출하기:

```ini
[mysqld]
# /etc/my.cnf에 추가
innodb_force_recovery=3  # 1~6: 복구 정도 단계

# 1: 무시할 수 없는 트랜잭션 건너뛰기
# 2: 모든 트랜잭션 건너뛰기
# 3: Undo Log 무시
# 4: Redo Log 무시
# 5: Insert Buffer 무시
# 6: Page 손상 무시 (가장 극단적)
```

```bash
# MySQL 시작 (읽기 전용)
systemctl start mysql

# 데이터 덤프
mysqldump --all-databases > /tmp/recovery_dump.sql

# 정상 복구 (my.cnf에서 innodb_force_recovery 제거)
systemctl stop mysql
vi /etc/my.cnf  # innodb_force_recovery 주석 처리

# 복구된 데이터 재입력
systemctl start mysql
mysql < /tmp/recovery_dump.sql
```

---

## 7. XtraBackup 주요 옵션

| 옵션 | 설명 |
|------|------|
| `--no-lock` | Lock 없이 온라인 백업 |
| `--incremental` | 증분 백업 모드 |
| `--incremental-basedir` | 증분 백업 기준 디렉토리 |
| `--apply-log` | 준비(prepare) 단계 실행 |
| `--redo-only` | Redo Log만 적용 (증분 중간 단계) |
| `--copy-back` | 복원(restore) |
| `--defaults-file` | my.cnf 경로 지정 |
| `--user / --password` | MySQL 백업 계정 |
| `--target-dir` | 백업 대상 디렉토리 |
| `--stream=xbstream` | Streaming 백업 (네트워크 전송 시 효율) |

---

## 8. 백업 자동화 스크립트

### 8.1 Daily Full + Weekly Incremental

```bash
#!/bin/bash
# /backup/backup.sh

BACKUP_BASE="/backup/mysql"
MYSQL_USER="backupuser"
MYSQL_PASSWORD="password"
RETENTION_DAYS=30

DAY_OF_WEEK=$(date +%w)  # 0=Sunday, 1=Monday, ...

if [ $DAY_OF_WEEK -eq 0 ]; then
    # Sunday: Full Backup
    BACKUP_DIR="$BACKUP_BASE/full_$(date +%Y%m%d)"
    innobackupex \
        --defaults-file=/etc/my.cnf \
        --no-lock \
        --user=$MYSQL_USER \
        --password=$MYSQL_PASSWORD \
        $BACKUP_DIR
    echo "Full backup: $BACKUP_DIR"
else
    # Weekday: Incremental
    FULL_BACKUP=$(ls -dt $BACKUP_BASE/full_* | head -1)
    BACKUP_DIR="$BACKUP_BASE/incr_$(date +%Y%m%d_%H%M%S)"
    innobackupex \
        --defaults-file=/etc/my.cnf \
        --incremental \
        --incremental-basedir=$FULL_BACKUP \
        --no-lock \
        --user=$MYSQL_USER \
        --password=$MYSQL_PASSWORD \
        $BACKUP_DIR
    echo "Incremental backup: $BACKUP_DIR"
fi

# 오래된 백업 정리
find $BACKUP_BASE -type d -mtime +$RETENTION_DAYS -exec rm -rf {} \;
```

### 8.2 Cron 스케줄

```bash
# /etc/cron.d/mysql_backup
0 2 * * * root /backup/backup.sh >> /var/log/mysql_backup.log 2>&1
```

---

## 9. 복구 성공 확인

```bash
# 1. MySQL 서비스 정상 시작 확인
systemctl status mysql

# 2. 데이터베이스 접속 및 쿼리 실행
mysql -u root -p -e "SELECT COUNT(*) FROM information_schema.tables;" mysql

# 3. InnoDB 상태 확인
mysql -u root -p -e "SHOW ENGINE INNODB STATUS;" | head -20

# 4. 데이터 일관성 확인
mysqlcheck -u root -p --all-databases
```

---

## 10. 연관 개념

- [[2026-06-13_(MySQL-Lock-Deadlock-모니터링)]]
- [[2026-06-13_(MySQL-Percona-설치-가이드)]]
- [[2026-06-13_(MySQL-Replication-가이드)]]
