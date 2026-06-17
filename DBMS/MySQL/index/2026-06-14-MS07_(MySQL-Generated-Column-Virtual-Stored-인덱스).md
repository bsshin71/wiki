# MySQL Generated Column (Virtual·Stored) + 인덱스

- **카테고리**: #DBMS #MySQL #index
- **태그**: #MySQL #index #generated-column #virtual #stored #REVERSE #튜닝
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-명시적생성칼럼+인덱스 - BigDataTeam]]

## 1. 핵심 요약

FBI+LIKE 버그를 우회하기 위해 **`GENERATED ALWAYS AS (REVERSE(col))` 생성 칼럼을 만들고 그 칼럼에 인덱스**를 건 뒤 **칼럼을 직접 조회**하면 `LIKE 'prefix%'`가 인덱스를 정상적으로 탄다(type=range). 저장 방식은 **VIRTUAL(저장 X, 인덱스 O)** 과 **STORED(저장 O)** 중 선택한다.

---

## 2. Virtual vs Stored

| 특징 | Virtual(인덱스 없음) | Virtual(인덱스 있음) | Stored |
|------|----------------------|----------------------|--------|
| 테이블 저장공간 | 없음 | 없음 | **있음** |
| 인덱스 저장공간 | 없음 | **있음** | **있음** |
| 검색속도 | 느림(전수계산) | **빠름** | **빠름** |
| 추천 | 가끔 조회 | 자주 조회/검색 | 계산비용 매우 클 때 |

## 3. 칼럼·인덱스 생성

```sql
-- Virtual (default)
ALTER TABLE t
  ADD COLUMN transaction_id_rev VARCHAR(15)
    GENERATED ALWAYS AS (REVERSE(transaction_id)) VIRTUAL,
  ADD INDEX idx_tid_rev2 (transaction_id_rev);

-- Stored
ALTER TABLE t
  ADD COLUMN transaction_id_rev VARCHAR(15)
    GENERATED ALWAYS AS (REVERSE(transaction_id)) STORED,
  ADD INDEX idx_tid_rev2 (transaction_id_rev);
```

## 4. 조회 (인덱스 사용 ✅)

```sql
ANALYZE TABLE t;
-- type=range, key=idx_tid_rev2
EXPLAIN SELECT * FROM t WHERE transaction_id_rev LIKE '9100%';
EXPLAIN SELECT * FROM t WHERE transaction_id_rev LIKE CONCAT(REVERSE('0019'),'%');
```
> **결론**: 생성 칼럼을 직접 조회하면 Virtual/Stored 모두 인덱스를 잘 탄다.

## 5. 연관 개념

- [[2026-06-14-MS06_(MySQL-Functional-Index-FBI-LIKE-한계)]] — 우회 대상 버그
