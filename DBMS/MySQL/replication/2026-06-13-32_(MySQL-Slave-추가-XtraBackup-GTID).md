# MySQL Slave 추가 — XtraBackup (GTID based)

- **카테고리**: #DBMS #MySQL #Replication
- **태그**: #replication #slave추가 #XtraBackup #GTID
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-slave 추가방법(gtid based) - BigDataTeam]]

## 1. 핵심 요약

XtraBackup으로 Master의 물리 백업을 받아 신규 Slave를 구성하는 GTID 기반 방법입니다.
백업본의 `xtrabackup_binlog_info`에서 GTID를 확인하여 `gtid_purged`로 설정한 뒤, `MASTER_AUTO_POSITION=1`로 복제를 시작합니다.

---

## 2. STEP 1 — 백업 준비 (Master)

prepare/copy-back을 위해 **압축 옵션 없이** 백업을 받습니다.

```bash
xtrabackup --defaults-file=${CNFFILE} --backup \
  --user=bkpuser --password=**** --target-dir=${BACKUPDIR}
```

```bash
# xtrabackup_binlog_info 에서 GTID 확인 (이후 gtid_purged 에 사용)
cat xtrabackup_binlog_info
# mysql-bin.000076   196   4074d152-0c4c-11eb-aa27-02f7979ac73c:1-1940334
```

prepare 후 qpress로 압축(`qp.sh`), 원본 삭제(`rm.sh`).

## 3. STEP 2 — 백업 전송 및 압축 해제 (Slave)

```bash
scp tsdb01.tar bos@10.1.5.22:/backup/mysql/fullbackup
sh decomp.sh   # qpress 압축 해제
```

## 4. STEP 3 — copy-back 으로 Slave DB 생성

```bash
./mysql.sh stop
mv data data.bak && mkdir data    # 기존 data 비우기
./copybak.sh                      # xtrabackup --copy-back
./mysql.sh start
```

---

## 5. STEP 4 — 복제 연결 및 시작

```sql
RESET MASTER;
-- gtid_purged 는 STEP 1의 xtrabackup_binlog_info 참조
SET GLOBAL gtid_purged="4074d152-0c4c-11eb-aa27-02f7979ac73c:1-1940334";
CHANGE MASTER TO
  MASTER_HOST="$masterip", MASTER_USER="repl",
  MASTER_PASSWORD="$slavepass", MASTER_AUTO_POSITION=1;
START SLAVE;
SHOW SLAVE STATUS \G
```

## 6. STEP 5 — 복제 상태 확인

```text
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Retrieved_Gtid_Set: 4074d152-...:1947268-1955257
Executed_Gtid_Set:  4074d152-...:1-1947821
```

---

## 7. 연관 개념

- [[2026-06-13-31_(MySQL-Slave-추가-CLONE-플러그인)]]
- [[2026-06-13-33_(MySQL-Slave-추가-mysqldump)]]
- [[2026-06-13_(MySQL-XtraBackup-백업복구-가이드)]]
- [[2026-06-13-02_(MySQL-GTID-기반-복제)]]
