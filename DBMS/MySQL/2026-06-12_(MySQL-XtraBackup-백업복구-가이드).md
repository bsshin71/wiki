# MySQL XtraBackup 백업·복구 가이드
- **카테고리**: #DBMS #MySQL
- **태그**: #MySQL #backup #XtraBackup #Percona #시점복구 #증분백업 #장애복구
- **작성일**: 2026-06-12
- **참조 원본**: [[2026-06-12-Xtrabackup Install setting - BigDataTeam]], [[2026-06-12-XtraBackup의 동작원리 - BigDataTeam]], [[2026-06-12-Xtrabackup 백업 요소 - BigDataTeam]], [[2026-06-12-Full 백업 및 복구 - BigDataTeam]], [[2026-06-12-증분 백업 - BigDataTeam]], [[2026-06-12-증분 백업 복구 테스트 - BigDataTeam]], [[2026-06-12-증분백업script(V8) - BigDataTeam]], [[2026-06-12-full백업script ( V8 ) - BigDataTeam]], [[2026-06-12-full백업script( v2.4 구버전용) - BigDataTeam]], [[2026-06-12-백업 복구 시나리오 - BigDataTeam]], [[2026-06-12-백업 복구 테스트 - BigDataTeam]], [[2026-06-12-시점 복구 - BigDataTeam]], [[2026-06-12-시점 복구 테스트 - BigDataTeam]], [[2026-06-12-긴급장애 복구 - BigDataTeam]]

## 1. 핵심 요약
- XtraBackup은 InnoDB **Hot Backup**(운영 중단 없이) 지원. 동작 원리: 데이터 파일 복사 + Redo Log 추적 → prepare 단계에서 roll-forward.
- 증분 백업 복구 시 마지막 이전 증분: `--apply-log-only --redo-only` 필수. 마지막 증분에서만 rollback 적용.
- **XtraBackup 8.0 → MySQL 8.0, Percona Server 8.0, XtraDB Cluster 8.0만 지원** (이전 버전 미지원).
- 긴급장애: `innodb_force_recovery=1~6` 단계적 적용 후 mysqldump로 데이터 추출.

## 2. 상세 설명

### XtraBackup 동작 원리

**InnoDB 백업 과정**:
1. 백업 시작 시점의 LSN(Log Sequence Number) 기록
2. 데이터 파일(`.ibd`) 물리 복사 시작
3. **백그라운드 프로세스**: Redo Log 감시 → 변경 내용을 `xtrabackup_logfile`에 아카이빙
4. 백업 완료 후 Non-InnoDB 파일 복사 시 `FLUSH TABLES WITH READ LOCK` 사용

**핵심**: 백업 중 변경된 데이터는 Redo Log에 따로 저장 → prepare 단계에서 **roll-forward**로 일관성 맞춤

**백업 결과 파일 설명**:

| 파일 | 내용 |
|------|------|
| `xtrabackup_binlog_info` | 백업 완료 시점의 binlog 파일명·포지션·GTID |
| `xtrabackup_checkpoints` | 백업 종류(full/incremental) + from_lsn, to_lsn, last_lsn |
| `xtrabackup_info` | 백업 시 사용된 옵션, 버전, 시작/종료 시간 |
| `xtrabackup_logfile` | 백업 중 변경된 Redo Log (이진 파일, 복구 필수) |
| `xtrabackup_tablespaces` | 테이블스페이스 정보 |

---

### XtraBackup 설치 및 백업 계정 설정

```bash
# Repository 설치
sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
sudo percona-release enable-only tools release
sudo yum install percona-xtrabackup-80
```

**버전별 백업 계정 권한**:

```sql
-- XtraBackup 8.0.35+ (최신)
CREATE USER 'bkpuser'@'localhost' IDENTIFIED WITH mysql_native_password BY '****';
GRANT BACKUP_ADMIN, PROCESS, RELOAD, LOCK TABLES, REPLICATION CLIENT,
      FLUSH_OPTIMIZER_COSTS, FLUSH_STATUS, FLUSH_TABLES, FLUSH_USER_RESOURCES
      ON *.* TO 'bkpuser'@'localhost';
GRANT SELECT ON performance_schema.log_status TO 'bkpuser'@'localhost';
GRANT SELECT ON performance_schema.keyring_component_status TO bkpuser@'localhost';
GRANT SELECT ON performance_schema.replication_group_members TO bkpuser@'localhost';
FLUSH PRIVILEGES;
```

> ⚠️ 버전별 필요 권한이 다름. `caching_sha2_password` 인증 오류 발생 시:
> `ALTER USER 'bkpuser'@'localhost' IDENTIFIED WITH mysql_native_password BY '****';`

---

### Full 백업 및 복구

**백업**:
```bash
xtrabackup --defaults-file=/etc/my.cnf \
  --user=bkpuser --password=*** \
  --backup \
  --parallel=4 \
  --compress \
  --compress-threads=2 \              # 8.0.35+: compress-threads (복수형)
  --target-dir=/backup/mysql/fullbackup/$(date +%Y-%m-%d)

# 백업 후 GTID 확인
cat /backup/mysql/fullbackup/날짜/xtrabackup_binlog_info
# binlog.000026  156  4074d152-...:1-1940334
```

**복구 절차**:
```bash
# 1. Decompress (압축 백업인 경우)
xtrabackup --defaults-file=/etc/my.cnf --user=bkpuser --password=*** \
  --decompress --target-dir=/backup/fullbackup/날짜

# 2. Prepare (apply-log: roll-forward)
xtrabackup --defaults-file=/etc/my.cnf \
  --prepare --target-dir=/backup/fullbackup/날짜

# 3. MySQL 종료 + 기존 데이터 백업
systemctl stop mysqld
mv /var/lib/mysql /var/lib/mysql-bak
mkdir /var/lib/mysql
chown -R mysql:mysql /var/lib/mysql

# 4. Copy-back
xtrabackup --defaults-file=/etc/my.cnf \
  --copy-back --target-dir=/backup/fullbackup/날짜
chown -R mysql:mysql /var/lib/mysql

# 5. MySQL 시작
systemctl start mysqld
```

**오류 대처**:
```bash
# 권한 오류 (SELinux)
setenforce 0
systemctl start mysqld

# socket 오류
systemctl stop mysqld; chmod 777 /var/lib/mysql; systemctl start mysqld

# copy-back 시 datadir이 비어있지 않으면
# Original data directory /var/lib/mysql is not empty!
rsync -avrP /backup/fullbackup/날짜/ /var/lib/mysql/
# 또는 mv 후 copy-back
```

---

### 증분 백업 및 복구

**증분 백업 전략 (LSN 기반)**:
```bash
# STEP 1: Full 백업
xtrabackup --defaults-file=/etc/my.cnf --user=bkpuser --password=*** \
  --backup --target-dir=/backup/fullbak/$(date +%Y-%m-%d)

# Full 백업의 마지막 LSN 확인
cat /backup/fullbak/날짜/xtrabackup_checkpoints
# backup_type = full-backuped
# from_lsn = 0
# to_lsn = 805715515
# last_lsn = 805715524

# STEP 2: 1차 증분 (이전 백업 경로 기준)
xtrabackup --defaults-file=/etc/my.cnf --user=bkpuser --password=*** \
  --backup \
  --incremental --incremental-basedir=/backup/fullbak/날짜 \
  --target-dir=/backup/inc1/$(date +%Y-%m-%d)

# STEP 3: 2차 증분 (1차 증분 기준)
xtrabackup --defaults-file=/etc/my.cnf --user=bkpuser --password=*** \
  --backup \
  --incremental --incremental-basedir=/backup/inc1/날짜 \
  --target-dir=/backup/inc2/$(date +%Y-%m-%d)
```

**LSN 방식 증분 백업 Script (V8)**:
```bash
#!/bin/bash
# fullbak4inc.sh - Full + lastbackupinfo 기록
BACKUPPATH=/backup/mysql/fullbackup
INCBACKUPPATH=/backup/mysql/increment
CNFFILE="/home/bos2/mysql/etc/my.cnf"
DATEDIR=`date "+%Y-%m-%d"`
BACKUPDIR="${BACKUPPATH}/${DATEDIR}"

mkdir -p $BACKUPDIR
xtrabackup --defaults-file=${CNFFILE} --backup --user=bkpuser --password=*** \
  --parallel=4 --compress --compress-threads=2 --target-dir=${BACKUPDIR}

# 다음 증분 백업 시 참조할 Last LSN 기록
echo "backuppath=${BACKUPDIR}" > ${INCBACKUPPATH}/lastbackupinfo
cat ${BACKUPDIR}/xtrabackup_checkpoints >> ${INCBACKUPPATH}/lastbackupinfo
```

```bash
#!/bin/bash
# incrementbak.sh - 증분 백업 (LSN 기반)
BACKUPPATH=/backup/mysql/increment
CNFFILE="/home/bos2/mysql/etc/my.cnf"
DATEDIR=`date "+%Y-%m-%d"`
BACKUPDIR="${BACKUPPATH}/${DATEDIR}"

mkdir -p $BACKUPDIR
LAST_LSN=`grep "last_lsn" ${BACKUPPATH}/lastbackupinfo | cut -d'=' -f2 | tr -d ' '`

xtrabackup --defaults-file=${CNFFILE} --backup --user=bkpuser --password=*** \
  --parallel=4 --compress --compress-threads=2 \
  --incremental-lsn=${LAST_LSN} \               # 이전 백업의 LAST LSN 이후부터
  --target-dir=${BACKUPDIR}

echo "backuppath=${BACKUPDIR}" > ${BACKUPPATH}/lastbackupinfo
cat ${BACKUPDIR}/xtrabackup_checkpoints >> ${BACKUPPATH}/lastbackupinfo
```

**증분 백업 복구 절차**:
```bash
# 1. 전체 백업 decompress
xtrabackup --decompress --target-dir=/backup/fullbak/날짜
xtrabackup --decompress --target-dir=/backup/inc1/날짜
xtrabackup --decompress --target-dir=/backup/inc2/날짜

# 2. Full 백업 prepare (redo-only: rollback 제외)
xtrabackup --apply-log-only --prepare \
  --target-dir=/backup/fullbak/날짜

# 3. 1차 증분 적용 (redo-only 필수 - 마지막 증분 전까지)
xtrabackup --apply-log-only --prepare \
  --target-dir=/backup/fullbak/날짜 \
  --incremental-dir=/backup/inc1/날짜

# 4. 마지막 증분 적용 (--redo-only 제거 → rollback 포함)
xtrabackup --prepare \
  --target-dir=/backup/fullbak/날짜 \
  --incremental-dir=/backup/inc2/날짜

# 5~8. MySQL 종료 → 데이터 이동 → copy-back → 시작 (Full 복구와 동일)
```

> ⚠️ **xtrabackup 8.0.22-15 이하 버그**: `--apply-log-only --prepare` 시 `Unknown error 3611` 발생
> → xtrabackup 8.0.23-16 이상으로 업그레이드 필요

---

### 시점 복구 (Point-in-Time Recovery)

백업 완료 시점 이후 특정 시점까지 복구:

```bash
# 1. Full 백업으로 복구 완료
# 2. 백업 시점의 Binlog position 확인
cat /backup/fullbak/날짜/xtrabackup_binlog_info
# bin_log.000001  154

# 3. mysqlbinlog으로 변경분 SQL 변환
mysqlbinlog bin_log.000001 --start-position=154 > 001.sql
mysqlbinlog bin_log.000002 > 002.sql
mysqlbinlog bin_log.000003 --stop-position=8564 > 003.sql   # 원하는 시점

# 4. 순차적으로 SQL 적용 (반드시 순서 지켜야 함)
mysql -uroot -p < 001.sql
mysql -uroot -p < 002.sql
mysql -uroot -p < 003.sql
```

**mysqlbinlog 주요 옵션**:

| 옵션 | 설명 |
|------|------|
| `--start-position=N` | N 이상의 포지션부터 읽기 시작 |
| `--stop-position=N` | N 직전까지만 읽기 |
| `--start-datetime=DATETIME` | 특정 시각부터 (`'2019-07-01 12:00:00'`) |
| `--stop-datetime=DATETIME` | 특정 시각까지 |
| `--database=DB명` | 특정 DB 관련 이벤트만 |

**특정 테이블만 복구 (EXPORT/IMPORT 방식)**:
```bash
# 백업에서 특정 테이블 파일 찾기
find /backup/fullbackup/날짜 -name "city.*"

# Prepare + Export
xtrabackup --prepare --export --target-dir=/backup/fullbackup/날짜

# MySQL에서 테이블 복구
mysql> ALTER TABLE city DISCARD TABLESPACE;
# cp city.cfg, city.ibd → datadir
mysql> ALTER TABLE city IMPORT TABLESPACE;
```

---

### 아카이브 Binary Log 백업 Script

```bash
#!/bin/bash
# archivelog.sh - 현재 binlog 파일들을 archive 경로로 복사 (증분 방식)
backupdir="/backup/mysql/archbinlog"
user=bkpuser; pass=***

mysqlbinlog=`mysql -u$user -p$pass -e "select @@log_bin_basename" --vertical 2>/dev/null \
  | grep log_bin_basename | tail -1 | awk '{print $2}'`
mysqlbinlog=`dirname $mysqlbinlog`

mysql -u$user -p$pass -e "show binary logs" 2>/dev/null | awk '{if(NR>=2) print $1}' | while read binlogfile
do
  orgfname=${mysqlbinlog}/${binlogfile}
  orgfsize=`stat -c%s ${mysqlbinlog}/${binlogfile}`
  archfname=${backupdir}/arch.${binlogfile}
  archfsize=`stat -c%s ${archfname} 2>/dev/null || echo 0`

  # 새 파일이거나 크기가 증가했으면 복사
  if [ "$archfsize" == 0 ] || [ $orgfsize -gt $archfsize ]; then
    cp $orgfname $archfname
  fi
done
```

**Crontab 설정**:
```cron
# 매일 새벽 4시 전체 백업 (UTC 19시 = KST 04시)
0 19 * * * /backup/mysql/script/fullbak.sh
# 10분마다 binlog 아카이빙
*/10 * * * * /backup/mysql/script/archivelog.sh 2>&1 >> /backup/mysql/trclog/archive.log
# 매일 오래된 백업 삭제
0 18 * * * /backup/mysql/script/rmoldbackup.sh 2>&1 >> /backup/mysql/trclog/rmoldbackup.log
```

---

### 긴급 장애 복구 (innodb_force_recovery)

MySQL 자동 복구 실패 시 `innodb_force_recovery` 값을 높여가며 강제 시작:

```ini
[mysqld]
innodb_force_recovery=1   # 1부터 시작, 시작 안 되면 값 증가
```

| 값 | 상황 | 권장 조치 |
|----|------|-----------|
| 1 (SRV_FORCE_IGNORE_CORRUPT) | 데이터/인덱스 페이지 손상. 로그에 "corruption on disk" | mysqldump 후 DB 재구축 |
| 2 (SRV_FORCE_NO_BACKGROUND) | undo 레코드 삭제 중 장애 | |
| 3 (SRV_FORCE_NO_TRX_UNDO) | 미커밋 TX 롤백 안 함 | mysqldump 후 재구축 |
| 4 (SRV_FORCE_NO_IBUF_MERGE) | Insert Buffer 손상 | |
| 5 (SRV_FORCE_NO_UNDO_LOG_SCAN) | Undo 로그 사용 불가. 미커밋도 커밋 처리됨 | mysqldump 후 재구축 |
| 6 (SRV_FORCE_NO_LOG_REDO) | Redo 로그 손상. 마지막 체크포인트 시점만 복구 | Redo 로그 삭제 후 재시작, mysqldump |

> ⚠️ 값이 커질수록 데이터 손실 가능성 증가. 최솟값부터 시작.
> 복구 후 반드시 mysqldump로 데이터 추출 → DB 재구축.

```bash
# Redo Log 손상 시 (값=6인 경우)
# Redo log 파일 이동 후 재시작
mv /var/lib/mysql/ib_logfile* /tmp/
systemctl start mysqld
# 시작 성공 시 즉시 mysqldump
mysqldump --all-databases -uroot -p > emergency_dump.sql
```

## 3. 연관 개념 (지식 연결)
- [[2026-06-12_(MySQL-Replication-가이드)]] — XtraBackup을 이용한 Slave 추가
- [[2026-06-12_(MySQL-Percona-설치-가이드)]] — XtraBackup 설치 환경 (Percona Server + 백업 계정)
