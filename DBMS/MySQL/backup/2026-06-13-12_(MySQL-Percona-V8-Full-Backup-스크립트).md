# MySQL Percona V8 Full Backup 스크립트

- **카테고리**: #DBMS #MySQL #Backup
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-full백업script ( V8 ) - BigDataTeam.md]]

## 1. 핵심 요약

Percona XtraBackup 8.0+(V8)은 **xtrabackup 명령**으로 통일되었으며, innobackupex는 deprecated 되었습니다.
V8의 백업 사용자는 BACKUP_ADMIN 권한이 필요하고, compress-thread→compress-threads로 옵션명이 변경되었습니다.
병렬 백업, 압축, 스트림 전송을 지원하여 대용량 DB의 효율적 백업을 가능하게 합니다.

---

## 2. V8 사용자 생성 (권한 변경)

```sql
CREATE USER 'bkpuser'@'localhost' IDENTIFIED BY 'password';
GRANT BACKUP_ADMIN, PROCESS, RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* 
TO 'bkpuser'@'localhost';
GRANT SELECT ON performance_schema.log_status TO 'bkpuser'@'localhost';
-- GRANT SELECT ON performance_schema.keyring_component_status (옵션)

ALTER USER 'bkpuser'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
FLUSH PRIVILEGES;
```

**V8 권한 변경 사항:**
- `BACKUP_ADMIN` 필수 추가 (구: RELOAD)
- `log_status` 테이블 접근 필수

---

## 3. Full Backup 스크립트 (Non-Stream)

```bash
#!/bin/bash

. /home/mysql/dbbackup/script/backup.cnf

if [ ! -d $BACKUPDIR ]; then
  mkdir -p $BACKUPDIR
else
  rm -rf ${BACKUPDIR}
fi

xtrabackup --defaults-file=${CNFFILE} \
  --backup \
  --user=${USER} \
  --password=${PASS} \
  --socket=${SOCKET} \
  --parallel=4 \
  --compress \
  --compress-threads=2 \
  --target-dir=${BACKUPDIR} >> ${TRCLOGFILE} 2>&1

# Tar 압축 (IO Intensive)
ionice -c 3 tar cvf ${BACKUPFILE} ${BACKUPDIR} >> ${TRCLOGFILE} 2>&1
ionice -c 3 mv ${BACKUPFILE} ${BACKARCHIVE_DIR}
```

**주요 옵션 (V8 변경사항):**
- `--compress-thread` → **`--compress-threads`** (복수형으로 변경, v8.0.35+)
- `--parallel=4`: 4개 스레드 병렬 백업
- `--compress`: 압축 활성화
- `ionice -c 3`: 낮은 우선순위 I/O (Replication Lag 방지)

---

## 4. Full Backup 스크립트 (Streaming)

```bash
xtrabackup --defaults-file=${CNFFILE} \
  --backup \
  --user=bkpuser \
  --password=xxxx \
  --parallel=4 \
  --compress \
  --compress-threads=2 \
  --stream=xbstream \
  --target-dir=./ > ${BACKUPDIR}/backup.xbstream
```

**Stream 장점:**
- 단일 파일로 완성 (통합 압축)
- 네트워크 전송 용이
- 디스크 I/O 감소

---

## 5. Archive Binlog 백업 스크립트

```bash
#!/bin/bash

backupdir="/backup/mysql/archbinlog"
user=bkpuser
pass=xxxx

# DB 연결 확인
PING=$(mysqladmin -u${user} -p${pass} ping 2>&1 | grep alive)
if [ "${PING}x" != "x" ]; then
  echo "DB is alive"
else
  echo "DB is Dead!!!!"
  exit 1
fi

# Binlog 경로 동적 조회
mysqlbinlog=$(mysql -u$user -p$pass -e "select @@log_bin_basename" \
  --vertical 2>/dev/null | grep log_bin_basename | awk '{print $2}')
mysqlbinlog=$(dirname $mysqlbinlog)

# 각 Binlog 파일 복사
mysql -u$user -p$pass -e "show binary logs" 2>/dev/null | \
  awk '{if(NR >= 2) print $1}' | while read binlogfile; do
  
  orgfname=${mysqlbinlog}/${binlogfile}
  orgfsize=$(stat -c%s ${mysqlbinlog}/${binlogfile})
  
  archfname=${backupdir}/arch.${binlogfile}
  
  if [ -f $archfname ]; then
    archfsize=$(stat -c%s ${archfname})
  else
    archfsize=0
  fi
  
  # 새 파일 또는 변경된 파일 복사
  if [ "$archfsize" == 0 ] || [ $orgfsize -gt $archfsize ]; then
    echo "copy $orgfname to $archfname"
    cp $orgfname $archfname
  fi
done
```

---

## 6. 복구 절차

### 6.1 압축 해제

```bash
xtrabackup --defaults-file=/etc/my.cnf \
  --decompress \
  --target-dir=/backup/2021-01-28
```

### 6.2 Prepare (구 --apply-log)

```bash
xtrabackup --defaults-file=/etc/my.cnf \
  --prepare \
  --target-dir=/backup/2021-01-28
```

### 6.3 Copy-back

```bash
xtrabackup --defaults-file=/etc/my.cnf \
  --copy-back \
  --target-dir=/backup/2021-01-28
```

---

## 7. Crontab 설정 예

```bash
# Full Backup (매일 19:00 UTC = 04:00 KST)
0 19 * * * /backup/mysql/script/fullbak.sh

# Archive Binlog (10분마다)
*/10 * * * * /backup/mysql/script/archivelog.sh 2>&1 >> /backup/mysql/archbinlog.log

# 오래된 백업 삭제 (매일 18:00, 1일 이상)
0 18 * * * /backup/mysql/script/rmoldbackup.sh
```

---

## 8. V8 vs v2.4 비교

| 항목 | V8 (xtrabackup) | v2.4 (innobackupex) |
|------|----------------|-------------------|
| 명령어 | xtrabackup | innobackupex |
| 사용자 권한 | BACKUP_ADMIN | RELOAD |
| 압축옵션 | --compress-threads | --compress-thread |
| Prepare | --prepare | --apply-log |
| 권장 대상 | MySQL 8.0+ | MySQL 5.6~5.7 |

---

## 9. 연관 개념

- [[2026-06-13-11_(MySQL-Full-Backup-복구-innobackupex)]]
- [[2026-06-13-13_(MySQL-Percona-v2.4-Legacy-Backup-스크립트)]]
