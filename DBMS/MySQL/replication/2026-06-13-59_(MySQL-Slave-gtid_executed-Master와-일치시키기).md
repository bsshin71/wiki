# MySQL Slave gtid_executed를 Master와 일치시키기

- **카테고리**: #DBMS #MySQL #replication
- **태그**: #replication #GTID #gtid_purged #reset_master
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-slave 의 gtid_executed 를 master와 일치시키기 - BigDataTeam]]

## 1. 핵심 요약

Slave의 `gtid_executed`가 Master와 어긋났을 때, **`RESET MASTER` → `SET GLOBAL gtid_purged='Master의 값'`**으로 수동 동기화한 뒤 `CHANGE MASTER TO ... AUTO_POSITION=1`로 복제를 재연결합니다.
⚠️ `RESET MASTER`는 binary log를 모두 삭제하고, `gtid_purged`는 한 번 설정하면 변경 불가하므로 신중히 진행합니다.

---

## 2. 절차

### 2.1 현재 GTID 상태 확인
```sql
-- Master
SHOW MASTER STATUS\G                          -- Executed_Gtid_Set 확인
-- Slave
SHOW SLAVE STATUS\G
SHOW GLOBAL VARIABLES LIKE 'gtid_executed';   -- Master와 다르면 동기화 진행
```

### 2.2 Slave 복제 중지
```sql
STOP SLAVE;
```

### 2.3 GTID 수동 설정 (⚠️ 주의)
```sql
RESET MASTER;
SET GLOBAL gtid_purged = 'MASTER의_GTID_EXECUTED_값';
```
> - `RESET MASTER` → 모든 binary log 삭제. **실행 전 백업 필수.**
> - `gtid_purged`는 한번 설정 시 재변경 불가 → 신중히 입력.

### 2.4 복제 재설정
```sql
CHANGE MASTER TO
  MASTER_HOST='Master_IP',
  MASTER_USER='replication_user',
  MASTER_PASSWORD='replication_password',
  MASTER_AUTO_POSITION=1;
START SLAVE;
```

### 2.5 정상 동작 확인
```sql
SHOW SLAVE STATUS\G
-- Slave_IO_Running=Yes, Slave_SQL_Running=Yes 이면 정상
```

---

## 3. 연관 개념

- [[2026-06-13-02_(MySQL-GTID-기반-복제)]]
- [[2026-06-13-34_(MySQL-Replication-재설정)]]
- [[2026-06-13-58_(MySQL-멀티-소스-복제-구성)]]
- [[2026-06-13-06_(MySQL-Replication-에러-처리-및-복구)]]
