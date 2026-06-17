# MySQL mysqlbinlog 유틸 사용법

- **카테고리**: #DBMS #MySQL #binlog
- **태그**: #MySQL #binlog #mysqlbinlog #PITR #복구 #tools
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-mysqlbinlog - BigDataTeam]]

## 1. 핵심 요약

`mysqlbinlog`는 **binlog 추적·PITR 복구·실시간 아카이빙**에 사용하는 MySQL 공식 유틸. ROW 포맷 binlog를 사람이 읽을 수 있는 SQL로 변환하고, 시간(`--start-datetime`)·위치(`--start-position`) 범위 필터링 후 DB에 직접 적용할 수 있다.

---

## 2. 주요 사용법

### binlog 내용 확인 (ROW 포맷 → 가독성)
```bash
mysqlbinlog --base64-output=DECODE-ROWS -v arch.bin.000157 | more
mysqlbinlog --base64-output=DECODE-ROWS -vv arch.bin.000157  # 칼럼 타입·nullable 추가
```

### 특정 시간대 조회
```bash
mysqlbinlog --start-datetime="2005-12-25 11:25:56" \
            --stop-datetime="2005-12-25 12:00:00" binlog.000003
```

### 특정 오프셋(position) 범위 조회
```bash
mysqlbinlog --start-position=462692185 --stop-position=462693000 \
  --base64-output=DECODE-ROWS -v /var/lib/mysql/binlog.000001
# SQL에서도 확인 가능
SHOW BINLOG EVENTS IN 'binlog.000001' FROM 462692185 LIMIT 10;
```

### PITR 복구용 SQL 추출·적용
```bash
mysqlbinlog --skip-gtids binlog.000001 > /tmp/dump.sql
mysqlbinlog --skip-gtids binlog.000002 >> /tmp/dump.sql
mysql -u root -p -e "source /tmp/dump.sql"
```

### 실시간 아카이빙
```bash
mysqlbinlog --read-from-remote-server --raw --stop-never \
  --host=127.0.0.1 --port=3306 --user=root --password=**** \
  --result-file=/mnt/dbbackup/archbin/ bin.000156
```

## 3. binlog 이벤트 구조 이해

```
# at 126                  ← 오프셋 126에서 시작
#250324 23:20:24           ← 이벤트 시각
server id 2               ← 서버 ID
end_log_pos 237           ← 이벤트 종료 위치
CRC32 0x3c5c69b1          ← 무결성 체크섬
Previous-GTIDs            ← GTID 정보
```

## 4. 연관 개념

- [[2026-06-14-MS16_(MySQL-binlog-아카이빙-방식-비교-cp-vs-mysqlbinlog)]]
- [[2026-06-14-MS17_(MySQL-binlog-archiver-daemon-스크립트)]]
- [[2026-06-14-MS19_(MySQL-MariaDB-Flashback-binlog-DML-원복)]]
- [[2026-06-13-44_(MySQL-시점-복구-PITR)]]
