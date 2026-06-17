# MySQL ERROR 1292 — Incorrect datetime value (sql_mode)

- **카테고리**: #DBMS #MySQL #troubleshooting
- **태그**: #MySQL #troubleshooting #error #sql_mode #datetime #NO_ZERO_DATE
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-ERROR 1292 (22007) Incorrect datetime value - BigDataTeam]]

## 1. 핵심 요약

`ERROR 1292 (22007): Incorrect datetime value: '0000-00-00 00:00:00'`는 `sql_mode`에 **`NO_ZERO_IN_DATE`·`NO_ZERO_DATE`** 가 활성화된 경우 `'0000-00-00 00:00:00'` 값 삽입 시 발생. MySQL 5.7+에서 기본 활성화되어 이전 버전에서 마이그레이션 시 자주 나타난다.

---

## 2. 원인

```sql
SHOW VARIABLES LIKE 'sql_mode';
-- NO_ZERO_IN_DATE, NO_ZERO_DATE 포함 시 '0000-00-00...' 값 거부
INSERT INTO T4 (c1, c2) VALUES (3, '0000-00-00 00:00:00');
-- ERROR 1292 (22007): Incorrect datetime value
```

## 3. 해결 — sql_mode에서 제거

```sql
-- 세션만 적용 (즉시)
SET SESSION sql_mode = 'ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';

-- 전역 적용 (재접속 후 적용)
SET GLOBAL sql_mode = 'ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';

-- 영구 적용 (my.cnf)
[mysqld]
sql_mode="ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION"
```

> `NO_ZERO_IN_DATE`·`NO_ZERO_DATE` 두 항목을 제거하면 `'0000-00-00 00:00:00'` 값 삽입 가능.

## 4. 연관 개념

- [[2026-06-14-MS33_(MySQL-MY-010288-Connection-Attributes-Truncated)]]
