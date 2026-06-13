# MySQL Replication 재설정 (전체 재구성)

- **카테고리**: #DBMS #MySQL #Replication
- **태그**: #replication #재설정 #GTID #mysqldump
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-replication 재설정 방법 - BigDataTeam]]

## 1. 핵심 요약

복제가 깨졌거나 처음부터 다시 구성할 때, Master/Slave의 GTID를 **모두 리셋**한 뒤 mysqldump로 데이터를 재동기화하는 10단계 절차입니다.
핵심은 Slave에서 `RESET MASTER`로 `gtid_executed`를 비운 뒤 덤프를 적재하고 `MASTER_AUTO_POSITION=1`로 복제를 시작하는 것입니다.

---

## 2. 전체 절차 (10단계)

### Master 측

```sql
-- 1. master 리셋 및 lock
RESET MASTER;
FLUSH TABLES WITH READ LOCK;

-- 2. master 현재 상태 기록
SHOW MASTER STATUS;        -- 결과 기록

-- 3. 전체 덤프 (GTID 포함)
-- (별도 셸에서) mysqldump -uroot -p**** --all-databases --set-gtid-purged=ON > /home/work/all.sql

-- 4. lock 해제
UNLOCK TABLES;
```

### Slave 측

```sql
-- 5. slave 정지
STOP SLAVE;

-- 6. slave GTID 리셋 (gtid_executed / gtid_purged 가 비어야 함)
RESET MASTER;
SHOW GLOBAL VARIABLES LIKE 'gtid_executed';   -- 값 없음 확인
SELECT @@global.gtid_purged;                   -- 값 없음 확인

-- 7. 데이터 로드 (셸): mysql -uroot -p**** < all.sql

-- 8. 복제 설정
CHANGE MASTER TO
  MASTER_HOST='Master_Server_IP', MASTER_USER='repl',
  MASTER_PASSWORD='****', MASTER_PORT=3306, MASTER_AUTO_POSITION=1;

-- 9. 복제 시작
START SLAVE;
```

```sql
-- 10. 복제 테스트 (Master에서 생성 → Slave 반영 확인)
CREATE DATABASE repldb;
USE repldb;
CREATE TABLE t1(c1 CHAR(10));
INSERT INTO t1 VALUES('1');
```

---

## 3. 연관 개념

- [[2026-06-13-33_(MySQL-Slave-추가-mysqldump)]]
- [[2026-06-13-06_(MySQL-Replication-에러-처리-및-복구)]]
- [[2026-06-13-04_(MySQL-복제-명령어-모음)]]
