# MySQL XtraBackup 동작 원리

- **카테고리**: #DBMS #MySQL #Backup
- **태그**: #backup #XtraBackup #LSN #roll_forward
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-XtraBackup의 동작원리 - BigDataTeam]]

## 1. 핵심 요약

XtraBackup은 DB를 멈추지 않는 **Hot Backup** 도구입니다.
백업 중 변경분을 redo log로 추적해 별도 보관하고, 복구 시 `--prepare`로 **Roll Forward**하여 백업 최종 시점의 일관된 데이터로 만듭니다.
InnoDB는 무중단 복사, MyISAM 등 비트랜잭션 엔진은 `FLUSH TABLES WITH READ LOCK`으로 처리합니다.

---

## 2. 트랜잭션 엔진(InnoDB/XtraDB) 백업 과정

1. 백업 중 변경이 없으면 데이터 파일을 그대로 복사.
2. 백업 중 insert/update가 발생하면 **redo log를 추적해 변경분만 별도 영역(xtrabackup_logfile)에 보관**.
3. 각 데이터 파일의 백업 시점이 달라 그대로는 **불일치(inconsistent)** 상태.
4. 복구 시 보관된 redo log로 변경분을 적용해 **백업 최종 시점으로 통일** → **Roll Forward(롤 포워드)**.
   - `xtrabackup --prepare` 옵션 사용.

## 3. 비트랜잭션 엔진(MyISAM 등) 백업 과정

- 트랜잭션 미지원 → 모든 쿼리를 중지하고 물리 파일 복사.
- `FLUSH TABLES WITH READ LOCK` 사용.

---

## 4. 전체 흐름: 백업 → 로그 적용 → 복구

### 4.1 백업

```bash
xtrabackup --host="127.0.0.1" --port=3306 \
  --user=bkpuser --password=**** --default-file="/etc/my.cnf" \
  --backup --target-dir=./data/backups/
```

- 시작 시점 **LSN(Log Sequence Number)** 기록 → 복구 시 일관성 기준.
- 백그라운드 프로세스가 redo log를 감시하며 변경분을 xtrabackup_logfile에 백업.

```bash
# LSN 49709357 ~ 49709387 구간 백업 완료 의미
xtrabackup: Transaction log of lsn (49709357) to (49709387) was copied.
201222 08:55:35 completed OK!     # 정상 완료
```

### 4.2 로그 적용

- 아카이빙된(보관된) redo log를 적용. **복구 직전에 수행하는 것이 좋다.**

---

## 5. 연관 개념

- [[2026-06-13-36_(MySQL-XtraBackup-백업-요소-및-옵션)]]
- [[2026-06-13-37_(MySQL-XtraBackup-설치-및-백업-계정-권한)]]
- [[2026-06-13-38_(MySQL-XtraBackup-증분-백업-및-복구)]]
- [[2026-06-13-26_(MySQL-Redo-Log-및-로그버퍼-설정)]]
