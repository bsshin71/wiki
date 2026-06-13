# MySQL Slave 추가 — CLONE 플러그인

- **카테고리**: #DBMS #MySQL #Replication
- **태그**: #replication #slave추가 #CLONE #MySQL8
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-slave 추가방법(cone statement) - BigDataTeam]]

## 1. 핵심 요약

MySQL 8.0의 **CLONE 플러그인**은 Donor(원본)의 데이터를 Recipient(신규 Slave)로 물리 복제하여 빠르게 Slave를 구성합니다.
`CLONE INSTANCE` 한 번으로 전체 데이터 파일을 복사하고, 이후 GTID 기반 복제로 추가 변경분을 따라잡습니다.

---

## 2. Donor(원본) 설정

```sql
INSTALL PLUGIN CLONE SONAME "mysql_clone.so";

-- 보안상 전용 유저 생성 권장
CREATE USER clone_user IDENTIFIED BY "clone_password";
GRANT BACKUP_ADMIN ON *.* TO clone_user;
GRANT SELECT ON performance_schema.* TO clone_user;
GRANT EXECUTE ON *.* TO clone_user;
```

## 3. Recipient(신규 Slave) 설정

```sql
INSTALL PLUGIN CLONE SONAME "mysql_clone.so";
SET GLOBAL clone_valid_donor_list = "donor.host.com:3306";

CREATE USER clone_user IDENTIFIED BY "clone_password";
GRANT CLONE_ADMIN ON *.* TO clone_user;
GRANT SELECT ON performance_schema.* TO clone_user;
GRANT EXECUTE ON *.* TO clone_user;
```

---

## 4. CLONE 실행 및 진행 상태 확인

```sql
-- Recipient 에서 실행
CLONE INSTANCE FROM clone_user@10.1.5.61:3306 IDENTIFIED BY "clone_password";
```

```sql
-- clone 전체 상태
SELECT STATE, CAST(BEGIN_TIME AS DATETIME) AS "START TIME",
       sys.format_time(POWER(10,12)*(UNIX_TIMESTAMP(END_TIME)-UNIX_TIMESTAMP(BEGIN_TIME))) AS DURATION
FROM performance_schema.clone_status;
-- Completed | 2021-07-20 01:40:39 | 14.00 m

-- 단계별 progress (DROP DATA → FILE COPY → PAGE COPY → REDO COPY → FILE SYNC → RESTART → RECOVERY)
SELECT STAGE, STATE, CAST(BEGIN_TIME AS TIME) AS "START TIME"
FROM performance_schema.clone_progress;
```

> Donor 로그에서 `Stage progress: 100% completed` 후 RECOVERY까지 완료되면 클론 종료.

---

## 5. Replication 으로 추가 변경분 반영

```sql
CHANGE MASTER TO
  MASTER_HOST='10.1.5.61', MASTER_PORT=3306,
  MASTER_USER='repl', MASTER_PASSWORD='repl',
  MASTER_AUTO_POSITION=1;
START SLAVE;
```

⚠️ **UUID 충돌 오류** 시 (`Last_IO_Errno: 13117 ... equal MySQL server UUIDs`):
→ Slave의 `auto.cnf` 파일 삭제 후 재시작 (clone으로 복사되며 master와 UUID가 같아짐).

---

## 6. 연관 개념

- [[2026-06-13-32_(MySQL-Slave-추가-XtraBackup-GTID)]]
- [[2026-06-13-33_(MySQL-Slave-추가-mysqldump)]]
- [[2026-06-13-02_(MySQL-GTID-기반-복제)]]
