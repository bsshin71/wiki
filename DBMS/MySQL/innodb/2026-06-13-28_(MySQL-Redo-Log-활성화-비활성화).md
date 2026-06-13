# MySQL Redo Log 활성화 / 비활성화

- **카테고리**: #DBMS #MySQL #InnoDB
- **태그**: #InnoDB #redo_log #migration #대량적재
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-리두로그 활성화 및 비활성화 - BigDataTeam]]

## 1. 핵심 요약

대용량 데이터를 일괄 적재할 때 redo log를 일시 비활성화하면 적재 시간을 단축할 수 있습니다(MySQL 8.0+).
단, **비활성화 상태에서 비정상 종료 시 `innodb_force_recovery=6`으로만 재기동 가능**하므로 적재 직후 반드시 다시 활성화해야 합니다.

---

## 2. 비활성화 → 적재 → 활성화 절차

```sql
-- 1. redo log 기록 비활성화
ALTER INSTANCE DISABLE INNODB REDO_LOG;

-- 2. 대량 데이터 로딩
LOAD DATA INFILE '/tmp/bulk.csv' INTO TABLE 테이블명 ...;

-- 3. 적재 완료 후 활성화 (필수)
ALTER INSTANCE ENABLE INNODB REDO_LOG;
```

---

## 3. 활성 상태 확인

```sql
SHOW GLOBAL STATUS LIKE 'Innodb_redo_log_enabled';
-- +-------------------------+-------+
-- | Innodb_redo_log_enabled | OFF   |
-- +-------------------------+-------+
```

---

## 4. 주의 (⚠️ 장애 위험)

- `ALTER INSTANCE DISABLE INNODB REDO_LOG;` 실행 후 MySQL이 **비정상 종료**하면,
  이후 서버 시작을 위해 **`innodb_force_recovery = 6`** 설정 후 재기동해야 한다.
- 따라서 redo log 비활성화는 **일시적 대량 적재 구간에만** 사용하고, 적재 직후 즉시 활성화한다.

> 참고: Binary Log 활성/비활성은 [[2026-06-13-20_(MySQL-Binary-Log-활성화-비활성화)]] (별개 개념).

---

## 5. 연관 개념

- [[2026-06-13-26_(MySQL-Redo-Log-및-로그버퍼-설정)]]
- [[2026-06-13-10_(MySQL-Fast-Index-Creation-최적화)]]
- [[2026-06-13_(MySQL-XtraBackup-백업복구-가이드)]]
