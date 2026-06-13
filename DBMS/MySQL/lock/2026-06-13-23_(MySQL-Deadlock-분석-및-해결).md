# MySQL Deadlock 분석 및 해결

- **카테고리**: #DBMS #MySQL #Lock
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-mysql deadlock - BigDataTeam.md]]

## 1. 핵심 요약

Deadlock은 두 개 이상의 트랜잭션이 서로의 락을 기다릴 때 발생합니다.
InnoDB는 자동 감지하여 한 트랜잭션을 롤백하며, SHOW ENGINE INNODB STATUS로 분석합니다.

---

## 2. Deadlock 감지

```sql
SHOW ENGINE INNODB STATUS\G

-- 출력 중 Deadlock 섹션 확인:
-- LATEST DETECTED DEADLOCK
-- *** (1) TRANSACTION
-- *** (2) TRANSACTION
-- *** WE ROLL BACK TRANSACTION (1)
```

---

## 3. Deadlock 회피

1. 트랜잭션 짧게 유지
2. 일관된 순서로 락 획득
3. 재시도 로직 구현

```python
# 재시도 로직
for retry in range(3):
  try:
    execute_transaction()
    break
  except DeadlockError:
    time.sleep(0.1 * retry)
```

---

## 4. 연관 개념

- [[2026-06-13-16_(MySQL-Lock-개념-및-분석)]]
