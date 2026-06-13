# MySQL InnoDB Lock 모니터링

- **카테고리**: #DBMS #MySQL #Lock
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-mysql lock - BigDataTeam.md]]

## 1. 핵심 요약

INFORMATION_SCHEMA와 Performance Schema로 InnoDB 락 상태를 모니터링합니다.
Lock waits, held locks, conflict를 실시간 추적하여 성능 병목 제거합니다.

---

## 2. InnoDB Lock 모니터링

```sql
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;
SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;
```

---

## 3. 성능 스키마 활용

```sql
SELECT * FROM performance_schema.data_locks;
SELECT * FROM performance_schema.data_lock_waits;
```

---

## 4. 연관 개념

- [[2026-06-13-16_(MySQL-Lock-개념-및-분석)]]
