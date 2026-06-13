# MySQL XtraBackup 증분 백업 및 복구

- **카테고리**: #DBMS #MySQL #Backup
- **태그**: #backup #XtraBackup #증분백업 #incremental
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-증분 백업 - BigDataTeam]], [[2026-06-12-증분 백업 복구 테스트 - BigDataTeam]]

## 1. 핵심 요약

증분 백업은 직전 백업 이후 변경분(LSN 기준)만 백업하여 **용량을 크게 절감**하지만, 누적 복구로 **복구 시간이 길어질 수 있습니다**.
복구 시 마지막 증분을 제외한 모든 단계는 `--redo-only`로 롤 포워드만 수행하고, 마지막 증분에서만 `--redo-only`를 빼서 롤백까지 적용합니다.

---

## 2. 증분 백업 특징

- Full 대비 백업 시간 단축은 크지 않으나 **백업 용량은 크게 감소**.
- 누적 복구 필요 → 복구 시간 증가 가능.
- Full/증분 선택은 백업 시간 + DB 용량으로 판단.

---

## 3. 백업 시나리오 (예: 일 Full + 일별 증분)

```bash
# 1. 전체 백업 (일요일)
innobackupex --defaults-file=/etc/my.cnf --no-lock \
  --user=backupuser --password=*** /root/mysqlxtra/fullbak
# xtrabackup_checkpoints: backup_type=full-backuped, from_lsn=0, to_lsn=805715515

# 2. 1차 증분 (월) — basedir = 전체 백업 경로
innobackupex --user=backupuser --password=*** /root/mysqlxtra/inc1 \
  --incremental --incremental-basedir=/root/mysqlxtra/fullbak/2019-06-30_10-02-36/

# 3. 2차 증분 (화) — basedir = 1차 증분 경로
innobackupex --user=backupuser --password=*** /root/mysqlxtra/inc2 \
  --incremental --incremental-basedir=/root/mysqlxtra/inc1/2019-06-30_10-39-41
```

> 각 증분의 `from_lsn`은 이전 단계의 `to_lsn`에서 시작.

---

## 4. 증분 백업 복구 (⭐ --redo-only 규칙)

> **마지막 증분 직전까지는 `--redo-only`(롤 포워드만), 마지막 증분에서만 `--redo-only` 제외(롤백 포함).**
> 중간 단계에서 롤백하면 다음 증분의 트랜잭션 일관성이 깨짐.

```bash
# 1. 전체 백업 prepare (redo-only)
innobackupex --apply-log --redo-only /root/mysqlxtra/fullbak/2019-06-30_10-02-36

# 2. 1차 증분 적용 (redo-only)
innobackupex --apply-log --redo-only \
  /root/mysqlxtra/fullbak/2019-06-30_10-02-36/ \
  --incremental-dir=/root/mysqlxtra/inc1/2019-06-30_10-39-41/

# 3. 마지막(2차) 증분 적용 (redo-only 제거 → 롤백 포함)
innobackupex --apply-log \
  /root/mysqlxtra/fullbak/2019-06-30_10-02-36/ \
  --incremental-dir /root/mysqlxtra/inc2/2019-06-30_11-03-00

# 4. redo 로그 생성을 위한 전체 백업 재 prepare
innobackupex --apply-log /root/mysqlxtra/fullbak/2019-06-30_10-02-36

# 5~8. MySQL 중지 → 기존 datadir 백업/비우기 → copy-back → 권한 부여 → start
systemctl stop mysqld
mv /var/lib/mysql /var/lib/mysql-bak && mkdir /var/lib/mysql
innobackupex --copy-back /root/mysqlxtra/fullbak/2019-06-30_10-02-36
chown -R mysql:mysql /var/lib/mysql
systemctl start mysqld
```

---

## 5. 오류 대처

| 증상 | 대처 |
|------|------|
| `os_file_get_status() failed ... Can't determine file permissions` | `setenforce 0` 후 재시작 (SELinux) |
| `Can't connect ... socket '/var/lib/mysql/mysql.sock'` | 중지 → `chmod 777 mysql` → 재시작 |
| startup 실패 | `log-error` 경로 로그 확인. 대부분 **데이터파일 권한 문제** |

---

## 6. 자동화 (스크립트 + crontab)

```bash
# increbackup.sh 핵심 (압축·병렬·basedir 자동 결정)
xtrabackup --defaults-file=/etc/my.cnf --backup \
  --user=bkpuser --password=**** --parallel=4 --compress \
  --incremental-basedir=${BASEDIR} --target-dir=${INCREDIR}
```

```bash
# crontab 예시
0 19 * * *   /home/temp/scripts/fullbackup.sh    # 매일 full (UTC19=KST04)
5 *  * * *   /home/temp/scripts/increbackup.sh   # 매시 5분 증분
*/10 * * * * /home/temp/scripts/archivelog.sh    # 10분마다 archive log
0 18 * * *   /home/temp/scripts/rmoldbackup.sh   # 오래된 백업 정리
```

> ⚠️ 8.0.35+ 에서는 `--compress-thread` 사용 시 `WARNING: unknown option` → `--compress-threads` 로 변경.

---

## 7. 연관 개념

- [[2026-06-13-35_(MySQL-XtraBackup-동작원리)]]
- [[2026-06-13-36_(MySQL-XtraBackup-백업-요소-및-옵션)]]
- [[2026-06-13-11_(MySQL-Full-Backup-복구-innobackupex)]]
- [[2026-06-13-12_(MySQL-Percona-V8-Full-Backup-스크립트)]]
