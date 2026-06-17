# MySQL 이중화 에러 — ALTER COLUMN 데이터잘림 조치

- **카테고리**: #DBMS #MySQL #replication
- **태그**: #MySQL #replication #troubleshooting #slave-error #data-truncation #GTID-skip
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-이중화 에러 - BigDataTeam]]

## 1. 핵심 요약

`ALTER TABLE MODIFY COLUMN`으로 컬럼 길이를 줄일 때 slave에 기존 길이를 초과하는 데이터가 있으면 **Error 1265(Data truncated)** 가 발생해 복제가 중단된다. 조치는 ① 초과 데이터를 삭제 후 복제 재개, ② GTID `gtid_next`로 해당 트랜잭션 스킵, 두 가지다.

---

## 2. 에러 확인

```sql
SHOW SLAVE STATUS\G
-- Last_SQL_Errno: 1265
-- Last_SQL_Error: Worker 2 failed executing transaction ... Data truncated for column 'ACC_NO'

SELECT * FROM performance_schema.replication_applier_status_by_worker\G;
-- LAST_ERROR_NUMBER: 1265
-- Query: 'ALTER TABLE BOSRL.T_DEPO_MM MODIFY COLUMN ACC_NO char(9) ...'

-- 초과 데이터 확인
SELECT acc_no FROM BOSRL.T_DEPO_MM WHERE LENGTH(TRIM(acc_no)) > 9 LIMIT 10;
```

## 3. 조치

### 방법 A — 초과 데이터 삭제 후 복제 재개
```sql
STOP SLAVE;
DELETE FROM BOSRL.T_DEPO_MM WHERE LENGTH(TRIM(acc_no)) > 9;
START SLAVE;
```

### 방법 B — GTID로 트랜잭션 스킵
```sql
STOP SLAVE;
SET @@SESSION.GTID_NEXT = '4074d152-0c4c-11eb-aa27-02f7979ac73c:8125044';
BEGIN; COMMIT;   -- 빈 트랜잭션으로 해당 GTID 소비
SET SESSION GTID_NEXT = AUTOMATIC;
START SLAVE;
```
> DDL 스킵 시 schema 변경이 slave에 적용되지 않으므로 수동 DDL 실행 필요.

## 4. 연관 개념

- [[2026-06-13-06_(MySQL-Replication-에러-처리-및-복구)]] — 에러 처리 일반
- [[2026-06-13-02_(MySQL-GTID-기반-복제)]] — GTID 개념
- [[2026-06-14-MS23_(MySQL-GTID-consistency-위반-에러-처리)]]
