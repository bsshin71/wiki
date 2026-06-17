# MySQL 데이터 Load / Bulk Insert 성능 개선

- **카테고리**: #DBMS #쿼리튜닝
- **태그**: #MySQL #bulk_insert #LOAD_DATA #성능 #마이그레이션
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-데이터 Load 성능 올리기 - BigDataTeam]]

## 1. 핵심 요약

대량 적재(mysqldump 복원·마이그레이션) 시 **binlog/autocommit/제약조건 체크를 끄고, multi-value INSERT·PK 순서 적재·LOAD DATA**를 활용하면 속도를 크게 올릴 수 있습니다.
text 자료는 INSERT보다 `LOAD DATA`가 **20배 이상** 빠릅니다.

---

## 2. 적재 성능 개선 8가지

### 2.1 binlog OFF (세션)
```sql
SET SESSION sql_log_bin=OFF;
```
> ⚠️ Slave가 있으면 Slave도 동일 적재 작업을 별도로 해줘야 함(복제로 전파 안 됨).

### 2.2 autocommit OFF
```sql
SET autocommit=0;
-- ... import statements ...
COMMIT;
```

### 2.3 제약조건 체크 OFF
```sql
SET unique_checks=0;
SET foreign_key_checks=0;
-- ... import ...
SET unique_checks=1;
SET foreign_key_checks=1;
```

### 2.4 Multiple Insert (multi-value)
```sql
INSERT INTO yourtable VALUES (1,2),(5,5), ... ;   -- mysqldump 기본 형태
```

### 2.5 auto_increment 락 줄이기
```sql
SET innodb_autoinc_lock_mode=2;   -- interleaved
```

### 2.6 PK 순서로 Insert
- InnoDB는 clustered index → **PK 순서 적재 시 random I/O 감소**. (PK 없는 것보다 있는 게 유리)

### 2.7 LOAD DATA 사용
- text 자료는 INSERT 대비 **20배 이상 빠름**.

### 2.8 설정 변경 (적재 한정)
```sql
SET GLOBAL innodb_flush_log_at_trx_commit = 0;  -- 커밋 시 로그 flush 빈도 감소
SET GLOBAL innodb_log_buffer_size = xxx;
SET GLOBAL sort_buffer_size = xxx;
```

> ⚠️ 적재 후에는 내구성을 위해 `innodb_flush_log_at_trx_commit`를 원복(1) 권장.

---

## 3. 연관 개념

- [[2026-06-13-10_(MySQL-Fast-Index-Creation-최적화)]]
- [[2026-06-13-20_(MySQL-Binary-Log-활성화-비활성화)]]
- [[2026-06-13-26_(MySQL-Redo-Log-및-로그버퍼-설정)]]
- [[2026-06-13_(MySQL-Performance-Schema-활용)]]
