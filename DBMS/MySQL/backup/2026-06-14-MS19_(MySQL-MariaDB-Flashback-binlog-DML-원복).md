# MySQL MariaDB Flashback으로 DML 원복

- **카테고리**: #DBMS #MySQL #backup
- **태그**: #MySQL #backup #flashback #mariadb #binlog #DML복구 #PITR
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-MySQL에서 mariadb flashback 사용하기 - BigDataTeam]]

## 1. 핵심 요약

MySQL은 Flashback 기능이 없지만 **MariaDB의 mysqlbinlog(10.2.4+)에 포함된 `--flashback` 옵션**으로 binlog before image를 역전시켜 DML(DELETE/UPDATE/INSERT) 원복이 가능. 전제: `binlog_format=ROW`, `binlog_row_image=FULL`. GTID 환경은 복구 SQL에 `GTID_NEXT`를 수동 삽입해야 한다.

---

## 2. 전제 조건

```sql
SET GLOBAL binlog_row_image = FULL;
FLUSH BINARY LOGS;
-- 확인
SHOW VARIABLES LIKE '%binlog_row%';
```
> MariaDB mysqlbinlog 바이너리를 MySQL 서버에 복사: `cp ~/mariadb/bin/mysqlbinlog ~/binlog/mysqlbinlog.maria`

## 3. 복구 절차 (4단계)

### ① 실수 DML 확인 후 position 찾기
```sql
SHOW BINARY LOGS;
SHOW BINLOG EVENTS IN 'mysql-bin.000009';
-- 또는 변환 후 파일에서 확인
./mysqlbinlog.maria -vv --base64-output=DECODE-ROWS mysql-bin.000009 > mysql-bin.sql
```

### ② flashback SQL 생성
```bash
./mysqlbinlog.maria -d TESTDB -T t1 \
  --start-position=353 --stop-position=534 \
  --flashback mysql-bin.000009 > flashback.sql
```

### ③ GTID 처리 (GTID 모드 환경)
방법A — 최종 GTID+1을 직접 삽입:
```
SET @@SESSION.GTID_NEXT= '25343674-...:20'/*!*/;   ← COMMIT 전에 삽입
...COMMIT
SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /*!*/;        ← COMMIT 후 삽입
```
방법B — awk 스크립트로 임의 GTID 자동 삽입(실운영은 방법A 권장):
```bash
cat flashback.sql | awk 'BEGIN {"uuidgen -t" |& getline u} \
  /^DELIMITER \/*!*\/;/ { c+=1; print "SET @@SESSION.GTID_NEXT= \x27" u ":" c "\x27;" } \
  {print} /^\/\*!\*\/;/ { print "SET @@SESSION.GTID_NEXT=\x27AUTOMATIC\x27/*!*/;" }' > final.sql
```
> 임의 GTID 방법은 `gtid_executed` 값이 여러 개로 분리되는 단점 있음.

### ④ 복구 SQL 적용
```bash
mysql -uroot -p -D TESTDB < flashback.sql   # 또는 mysql -e "source flashback.sql"
```

## 4. 지원 범위

| 항목 | 내용 |
|------|------|
| 지원 버전 | MariaDB 10.2.4+ mysqlbinlog |
| 지원 DML | DELETE, UPDATE, INSERT |
| 미지원 | DDL(DROP, TRUNCATE, ALTER) — 향후 MariaDB 예정 |
| GTID 주의 | `ERROR 1782: GTID_NEXT cannot be ANONYMOUS when GTID_MODE=ON` 발생 시 GTID 처리 필요 |

## 5. 연관 개념

- [[2026-06-14-MS18_(MySQL-mysqlbinlog-유틸-사용법)]]
- [[2026-06-13-45_(MySQL-백업-복구-시나리오)]] — PITR 시나리오 전체
- [[2026-06-13-44_(MySQL-시점-복구-PITR)]]
