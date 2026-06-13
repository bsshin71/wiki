# MySQL Replication 에러 처리 및 복구

- **카테고리**: #DBMS #MySQL #Replication
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-07. replication 상황별 조치 - BigDataTeam.md]]

## 1. 핵심 요약

Slave 복제 중 SQL 에러(데이터 타입 변경, 제약조건 위반 등)가 발생하면 복제가 중단됩니다.
GTID 기반 복제에서는 문제 트랜잭션을 식별하고 `gtid_next` 로 건너뛰거나, Position 기반에서는 `sql_slave_skip_counter` 로 처리합니다.
복제 재개 전 에러 원인을 반드시 검토해야 데이터 불일치를 방지할 수 있습니다.

---

## 2. Replication 에러 감지

### 2.1 Slave 상태 확인

```sql
SHOW SLAVE STATUS\G
```

**주요 에러 필드**:
- `Slave_SQL_Running`: No → SQL 스레드 중단
- `Last_SQL_Errno`: 에러 코드 (0이 아니면 에러)
- `Last_SQL_Error`: 에러 메시지
- `Executed_Gtid_Set`: 마지막으로 성공한 GTID

**예시 에러**:
```
Last_Errno: 13146
Last_Error: Column1 of table 'DBBOS.T1' cannot be converted 
            from type 'varchar(40(bytes))' to type 'varchar(30(bytes) utf8)'
```

---

## 3. GTID 기반 복제에서 트랜잭션 건너뛰기

### 3.1 수동 GTID Skip (권장)

**Step 1: Slave 중단**

```sql
STOP SLAVE;
```

**Step 2: 문제 트랜잭션 식별**

```sql
-- SHOW SLAVE STATUS 에서 Executed_Gtid_Set 확인
-- 예: 4074d152-0c4c-11eb-aa27-02f7979ac73c:2
-- 다음 실행할 GTID는 :3 이 됨
```

**Step 3: 다음 GTID를 수동으로 건너뛰기**

```sql
SET SESSION gtid_next='4074d152-0c4c-11eb-aa27-02f7979ac73c:3';
BEGIN;
COMMIT;
```

**Step 4: GTID 모드 해제**

```sql
SET SESSION gtid_next='AUTOMATIC';
```

**Step 5: Slave 재시작**

```sql
START SLAVE;

-- 복제 정상화 확인
SHOW SLAVE STATUS\G
```

### 3.2 다중 에러 처리 (반복)

에러가 여러 개면 위 3.2~3.5 과정을 반복합니다:

```sql
-- 에러 확인
SHOW SLAVE STATUS\G

-- 문제 GTID 건너뛰기
STOP SLAVE;
SET SESSION gtid_next='<다음 GTID>';
BEGIN; COMMIT;
SET SESSION gtid_next='AUTOMATIC';
START SLAVE;

-- 다시 확인
SHOW SLAVE STATUS\G
```

---

## 4. Position 기반 복제에서 트랜잭션 건너뛰기

**주의**: GTID 모드에서는 `sql_slave_skip_counter` 사용 불가

```sql
STOP SLAVE;

-- 1개 이벤트 건너뛰기
SET GLOBAL sql_slave_skip_counter=1;

START SLAVE;

-- 상태 확인
SHOW SLAVE STATUS\G
```

---

## 5. 에러 원인 분석 및 대응

### 5.1 데이터 타입 변환 에러

```
Column 'col' cannot be converted from type 'varchar(40)' to 'varchar(30)'
```

**원인**: Master에서 컬럼을 축소했는데 Slave의 실제 데이터가 더 긺

**대응**:
1. Master 스키마 확인: `SHOW CREATE TABLE table_name`
2. Slave에서 실제 데이터 길이 확인: `SELECT MAX(CHAR_LENGTH(col)) FROM table`
3. Slave 스키마를 Master와 동기화하거나 데이터 정리
4. 필요 시 Master에서 먼저 데이터 정리 후 스키마 변경

### 5.2 중복 키 에러 (Duplicate Key)

```
Duplicate entry '123' for key 'PRIMARY'
```

**원인**: 
- Slave에 이미 같은 키 존재
- Master와 Slave의 데이터 불일치

**대응**:
1. Slave에서 중복 레코드 삭제
2. 또는 전체 Slave 재구성

### 5.3 제약조건 위반

```
Cannot add or update a child row: foreign key constraint fails
```

**원인**: Foreign Key 제약 미충족

**대응**:
1. Master 데이터의 참조 무결성 검토
2. Slave에서 대기 중인 부모 레코드 확인
3. 필요 시 Foreign Key 임시 비활성화

---

## 6. 전체 복제 재구성 (최후의 수단)

무수히 많은 에러가 발생하거나 복제 불신뢰 시:

```sql
-- Slave에서 실행
STOP SLAVE;
RESET SLAVE ALL;

-- Master 정보 재설정 (GTID 기반)
CHANGE MASTER TO
  MASTER_HOST='master_ip',
  MASTER_PORT=3306,
  MASTER_USER='repl_user',
  MASTER_PASSWORD='password',
  MASTER_AUTO_POSITION=1;

START SLAVE;

-- 검증
SHOW SLAVE STATUS\G
```

---

## 7. 예방 체크리스트

| 항목 | 조치 |
|------|------|
| **스키마 동기화** | Master 스키마 변경 시 Slave에 먼저 적용 |
| **데이터 일관성** | 주기적으로 pt-table-checksum 실행 |
| **Binary Log 보관** | Master의 binlog_expire_logs_days 충분히 설정 |
| **Slave Relay Log** | relay_log_recovery=ON 으로 설정 |
| **모니터링** | Slave 상태 주기적 점검 (Seconds_Behind_Master) |

---

## 8. 참고 자료

- Percona gtid-failover: https://www.percona.com/blog/2015/12/02/gtid-failover-with-mysqlslavetrx-fix-errant-transactions/
- MySQL GTID 복제 복구: https://www.percona.com/blog/2013/02/08/how-to-createrestore-a-slave-using-gtid-replication-in-mysql-5-6/

---

## 9. 연관 개념

- [[2026-06-13-01_(MySQL-Binary-Log-Position-복제)]]
- [[2026-06-13-02_(MySQL-GTID-기반-복제)]]
- [[2026-06-13-04_(MySQL-복제-명령어-모음)]]
