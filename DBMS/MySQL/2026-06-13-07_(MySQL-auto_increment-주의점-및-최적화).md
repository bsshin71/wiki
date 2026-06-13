# MySQL auto_increment 주의점 및 최적화

- **카테고리**: #DBMS #MySQL #Schema
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-auto_increment 의 사용시 주의점 - BigDataTeam.md]]

## 1. 핵심 요약

MySQL의 `AUTO_INCREMENT` 컬럼은 PK 또는 UNIQUE 속성에만 적용되며, 증가한 값은 절대 줄어들지 않습니다.
InnoDB에서는 Rollback 후에도 시퀀스 값이 증가하므로 채번 시 주의가 필요합니다.
Replication 환경에서는 Rollback으로 인한 채번값 누락이 Slave의 시퀀스 동기화를 깨뜨릴 수 있습니다.

---

## 2. AUTO_INCREMENT 기본 규칙

### 2.1 PK 또는 UNIQUE 속성 필수

```sql
-- ✅ 허용 (PK에 적용)
CREATE TABLE users (
  id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(50)
);

-- ✅ 허용 (UNIQUE에 적용)
CREATE TABLE products (
  sku INT NOT NULL AUTO_INCREMENT UNIQUE,
  name VARCHAR(100)
);

-- ❌ 불허 (일반 컬럼)
CREATE TABLE logs (
  seq_num INT AUTO_INCREMENT  -- PK/UNIQUE 없음 → 오류
);
```

### 2.2 증가한 값은 절대 줄어들 수 없음

```sql
-- 초기 AUTO_INCREMENT=1로 생성
CREATE TABLE AUTO_INCREMENT_TEST (
  DA_NO INT NOT NULL AUTO_INCREMENT,
  DA_ID VARCHAR(11) NOT NULL,
  PRIMARY KEY(DA_NO, DA_ID)
) ENGINE=InnoDB AUTO_INCREMENT=1;

-- 2개 행 삽입 후 AUTO_INCREMENT는 3
INSERT INTO AUTO_INCREMENT_TEST (DA_ID) VALUES ('A');  -- DA_NO=1
INSERT INTO AUTO_INCREMENT_TEST (DA_ID) VALUES ('A');  -- DA_NO=2

-- AUTO_INCREMENT를 1로 재설정 시도
ALTER TABLE AUTO_INCREMENT_TEST AUTO_INCREMENT=1;
-- 결과: 무시됨 (이미 2개 행이 존재하므로)

-- 다음 삽입은 DA_NO=3이 됨
INSERT INTO AUTO_INCREMENT_TEST (DA_ID) VALUES ('B');  -- DA_NO=3 (1이 아님!)
```

---

## 3. InnoDB 재시작 후 AUTO_INCREMENT 동작

### 3.1 DB 재시작 시 초기화

InnoDB는 MySQL 서버 시작 후 **실제 데이터의 MAX값 + 1**로 AUTO_INCREMENT를 재설정합니다.

```sql
-- 현재 상태
SELECT * FROM table;
-- 결과: id=1, id=2, id=5 (id=3,4는 Rollback)

-- DB 재시작 후
SHOW CREATE TABLE table\G
-- AUTO_INCREMENT = 6 (MAX(id)+1)
```

### 3.2 Rollback과 AUTO_INCREMENT

```sql
-- Transaction A
BEGIN;
INSERT INTO seq_test VALUES (NULL);  -- AUTO_INCREMENT 증가 (예: id=10)
ROLLBACK;  -- 데이터는 롤백되지만 AUTO_INCREMENT는 그대로

-- Transaction B (A의 Rollback 후)
SELECT last_insert_id();  -- 10 (감소하지 않음)
BEGIN;
INSERT INTO seq_test VALUES (NULL);  -- id=11 (id=10 건너뜀)
```

---

## 4. 트랜잭션 내 채번 테스트

### 4.1 Stored Function을 이용한 채번

```sql
DELIMITER //
CREATE FUNCTION GetSeq2() RETURNS INT UNSIGNED
NOT DETERMINISTIC
BEGIN
    INSERT INTO SEQ_TEST VALUES (NULL);
    RETURN LAST_INSERT_ID();
END //
DELIMITER ;

CREATE TABLE SEQ_TEST (
  SEQ INT UNSIGNED NOT NULL AUTO_INCREMENT PRIMARY KEY
) ENGINE=InnoDB;
```

### 4.2 트랜잭션 간 채번값 비교 결과

| 세션 A | 세션 B | 테이블 내용 | 주요 발견 |
|--------|--------|----------|---------|
| BEGIN; SELECT GetSeq2() → 1 | (아직 시작 안 함) | Empty | - |
| - | BEGIN; SELECT GetSeq2() → 2 | Empty | Rollback 전까지 데이터 미반영 |
| COMMIT; | SELECT GetSeq2() → 3 | SEQ=1 | 세션B 채번(2)는 반영 안 됨 |
| - | COMMIT; | SEQ=1, SEQ=3 | - |

**결론:**
- **Commit/Rollback 상관없이 `LAST_INSERT_ID()` 값은 항상 증가**
- **실제 테이블에는 Commit된 값만 반영**
- **Rollback된 채번값은 영구적으로 건너뜀**

---

## 5. Replication 환경에서의 주의점

### 5.1 Rollback 후 채번값 미동기화

**Master → Slave Replication의 문제:**

| 순서 | Master (A) | Slave (B) | 비고 |
|------|-----------|----------|------|
| 1 | BEGIN; SELECT GetSeq() → 1 | (동기화 안 됨) | - |
| 2 | SELECT GetSeq() → 2 | - | - |
| 3 | SHOW CREATE TABLE → AUTO_INCREMENT=3 | SHOW CREATE TABLE → AUTO_INCREMENT=1 | **불일치!** |
| 4 | ROLLBACK; | - | - |
| 5 | SHOW CREATE TABLE → AUTO_INCREMENT=3 | SHOW CREATE TABLE → AUTO_INCREMENT=1 | 여전히 불일치 |
| 6 | BEGIN; SELECT GetSeq() → 3; COMMIT; | - | - |
| 7 | SHOW CREATE TABLE → AUTO_INCREMENT=4 | SHOW CREATE TABLE → AUTO_INCREMENT=4 | 이제 동기화됨 |

**문제**: Rollback된 채번값(1, 2)는 Slave로 복제되지 않으므로, **Failover 시 Slave가 Master보다 작은 채번값부터 시작**

### 5.2 Failover 시나리오

```
Master (id=100) 장애 발생
    ↓
Slave (id=50) 승격 → 새로운 Master
    ↓
id=51부터 시작 (id=51~100 중복 위험!)
```

---

## 6. 해결 방법

### 6.1 애플리케이션 레벨 개선

```java
// ❌ 안 좋은 패턴
begin transaction
  call getSeq(@num);
  insert into log_tbl(@num);  // getSeq와 insert가 같은 트랜잭션
commit;

// ✅ 좋은 패턴
begin transaction
  call getSeq(@num);
commit;  // 채번 먼저 커밋

begin transaction
  insert into log_tbl(@num);  // 별도 트랜잭션에서 사용
commit;
```

### 6.2 DB 시작 시 시퀀스 재조정

```sql
-- startup script (MySQL 시작 후 실행)
ALTER TABLE seq_table AUTO_INCREMENT = 
  (SELECT MAX(seq_column) + 1 FROM seq_table);
```

### 6.3 Replication 전략

**다중 Master 환경에서 채번 분산:**

```sql
-- Master 1
SET auto_increment_increment = 2;
SET auto_increment_offset = 1;  -- 1, 3, 5, 7, ...

-- Master 2
SET auto_increment_increment = 2;
SET auto_increment_offset = 2;  -- 2, 4, 6, 8, ...
```

---

## 7. 체크리스트

| 항목 | 점검 내용 |
|------|---------|
| **채번 테이블 설계** | ✅ PK 또는 UNIQUE 속성 확인 |
| **Rollback 처리** | ✅ 채번 트랜잭션을 별도로 분리 |
| **Replication 동기화** | ✅ Slave AUTO_INCREMENT 주기적 검증 |
| **Failover 준비** | ✅ 채번값 범위 미리 예약 (offset 사용) |
| **DB 재시작** | ✅ startup script로 AUTO_INCREMENT 재조정 |

---

## 8. 참고 자료

- MySQL AUTO_INCREMENT: https://dev.mysql.com/doc/refman/5.6/en/alter-table.html
- auto_increment 기초 개념: https://adbancedteam.tistory.com/149
- Replication 채번 전략: https://mozi.tistory.com/421

---

## 9. 연관 개념

- [[2026-06-13-04_(MySQL-복제-명령어-모음)]]
- [[2026-06-13-05_(MySQL-Admin-쿼리-기초)]]
