# MySQL Character Set 설정

- **카테고리**: #DBMS #MySQL #Schema
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-mysql character set 설정 - BigDataTeam.md]]

## 1. 핵심 요약

MySQL의 Character Set 불일치는 데이터 손상 및 복제 오류를 초래합니다.
Server, Database, Table, Column 레벨에서 개별 설정 가능하며, **utf8mb4 권장**입니다 (emoji, 다국어 지원).

---

## 2. Character Set 확인

```sql
-- 서버 기본값
SHOW VARIABLES LIKE 'character_set%';
SHOW VARIABLES LIKE 'collation%';

-- DB 별 설정
SELECT SCHEMA_NAME, DEFAULT_CHARACTER_SET_NAME FROM information_schema.SCHEMATA;

-- 테이블 별
SHOW CREATE TABLE table_name;
```

---

## 3. Character Set 변경

```sql
-- 서버
SET GLOBAL character_set_server = 'utf8mb4';

-- DB
ALTER DATABASE mydb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 테이블
ALTER TABLE mytable CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 연결
SET SESSION character_set_client = 'utf8mb4';
SET SESSION character_set_connection = 'utf8mb4';
SET SESSION character_set_results = 'utf8mb4';
```

---

## 4. my.cnf 설정

```ini
[mysqld]
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[client]
default-character-set = utf8mb4
```

---

## 5. 주요 Character Set

| 문자셋 | 바이트 | 용도 |
|--------|--------|------|
| utf8 | 3 | 다국어 (emoji 미지원) |
| **utf8mb4** | 4 | **권장** (emoji 지원) |
| latin1 | 1 | 영문 (레거시) |
| binary | 1 | 바이너리 |

---

## 6. 복제 환경에서 주의

Master/Slave의 character_set이 다르면 복제 오류 발생. **반드시 동일하게 설정**.

---

## 7. 연관 개념

- [[2026-06-13-01_(MySQL-Binary-Log-Position-복제)]]
