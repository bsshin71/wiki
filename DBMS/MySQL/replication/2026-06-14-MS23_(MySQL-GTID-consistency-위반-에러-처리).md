# MySQL GTID consistency 위반 에러 처리

- **카테고리**: #DBMS #MySQL #replication
- **태그**: #MySQL #replication #GTID #ERROR-1785 #MyISAM #troubleshooting
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Statement violates GTID consistency - BigDataTeam]]

## 1. 핵심 요약

`GTID_MODE=ON` 환경에서 **InnoDB(트랜잭션) + MyISAM(비트랜잭션) 테이블에 동일 트랜잭션 내 UPDATE** 시 `ERROR 1785: Statement violates GTID consistency`가 발생한다. 해결은 스토리지 엔진이 다른 테이블의 변경을 **별도 트랜잭션으로 분리**하는 것이다. `ENFORCE_GTID_CONSISTENCY=OFF`는 `GTID_MODE=ON`에서 설정 불가.

---

## 2. 재현

```sql
CREATE TABLE A1 (c1 INT) ENGINE=InnoDB;
CREATE TABLE M1 (c1 INT) ENGINE=MyISAM;

BEGIN;
INSERT INTO A1 VALUES (3);
INSERT INTO M1 VALUES (2);   -- ERROR 1785: Statement violates GTID consistency
```

## 3. 원인

GTID 환경에서 비트랜잭션(MyISAM) 테이블 업데이트는 **단독 autocommit 또는 단일 문장 트랜잭션**으로만 허용. 트랜잭션 엔진(InnoDB)과 같은 트랜잭션 내에서 혼용 불가.

## 4. 해결 — 트랜잭션 분리

```sql
BEGIN; INSERT INTO A1 VALUES (3); COMMIT;
BEGIN; INSERT INTO M1 VALUES (3); COMMIT;   -- 별도 트랜잭션
```

## 5. 기타 시도 (불가)

```sql
SET GLOBAL ENFORCE_GTID_CONSISTENCY = OFF;
-- ERROR 1779: GTID_MODE = ON requires ENFORCE_GTID_CONSISTENCY = ON
-- GTID 이중화 환경에서 GTID_MODE 해제 불가
```

## 6. 연관 개념

- [[2026-06-13-02_(MySQL-GTID-기반-복제)]]
- [[2026-06-14-MS22_(MySQL-이중화-에러-ALTER-COLUMN-데이터잘림-조치)]]
