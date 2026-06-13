# MySQL XtraBackup 백업 요소 및 옵션

- **카테고리**: #DBMS #MySQL #Backup
- **태그**: #backup #XtraBackup #옵션 #메타파일
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-Xtrabackup 백업 요소 - BigDataTeam]]

## 1. 핵심 요약

XtraBackup은 백업 시 데이터 파일과 함께 **메타데이터 파일**(checkpoints·binlog_info·info 등)을 생성하며, 이들은 복구·증분 백업의 기준이 됩니다.
주요 옵션(`--backup`, `--prepare`, `--copy-back`, `--apply-log-only`)의 용도를 정확히 이해해야 안전한 복구가 가능합니다.

---

## 2. 백업 생성 메타파일

| 파일 | 내용 |
|------|------|
| `xtrabackup_binlog_info` | 백업에 사용된 binary log 파일·position |
| `xtrabackup_checkpoints` | 백업 종료 시 LSN (`to_lsn`), backup_type(full/incremental) |
| `xtrabackup_info` | 백업 시각·사용 옵션·버전 등 상태 |
| `xtrabackup_logfile` | 백업용 redo log (이진 파일) |
| `xtrabackup_tablespaces` | 테이블스페이스 목록 |

```bash
# xtrabackup_checkpoints 예시
backup_type = full-backuped
from_lsn = 0
to_lsn = 49854072       # 증분 백업의 basedir 기준이 됨
last_lsn = 49854082
```

> binary log는 **백업 시점 직전 파일 일부만** 포함(전체를 백업하지 않음).

---

## 3. 주요 옵션

| 옵션 | 내용 |
|------|------|
| `--backup` | 백업 모드 |
| `--target-dir` | 백업 파일 저장 디렉터리 |
| `--user` / `--password` | 백업 권한 계정 |
| `--host` / `--port` / `--socket` | 접속 방식 |
| `--prepare` | **복구 모드**. ① 백업 중 변경분 적용(logfile→롤 포워드) ② innodb 로그 파일 재생성(backup-my.cnf 기준) |
| `--copy-back` | 백업 파일을 남기고 datadir로 복사 |
| `--move-back` | 백업 파일을 남기지 않고 datadir로 이동 |
| `--apply-log-only` | **증분 복구 시 필수**. 커밋 트랜잭션의 롤 포워드만 수행, 롤백 제외 (마지막 증분 직전까지 적용). 미사용 시 증분 데이터 연속성 깨질 수 있음 |
| `--stats` | 스캔 모드 |

---

## 4. 복구 준비 (prepare)

```bash
xtrabackup --prepare --target-dir=/home/bos/data/backups/2020-12-23
# xtrabackup_logfile detected ... crash recovery ... completed OK!
```

```bash
# copy-back: datadir 이 비어있어야 함
xtrabackup --copy-back --target-dir=/home/bos/data/backups/2020-12-23
# Original data directory /var/lib/mysql is not empty!  → datadir 비우거나 rsync 사용
```

---

## 5. 버전 호환

- **XtraBackup 8.0**은 MySQL 8.0 / Percona Server 8.0 / Percona XtraDB Cluster 8.0 만 지원 (이전 버전 미지원).

---

## 6. 연관 개념

- [[2026-06-13-35_(MySQL-XtraBackup-동작원리)]]
- [[2026-06-13-38_(MySQL-XtraBackup-증분-백업-및-복구)]]
- [[2026-06-13-11_(MySQL-Full-Backup-복구-innobackupex)]]
