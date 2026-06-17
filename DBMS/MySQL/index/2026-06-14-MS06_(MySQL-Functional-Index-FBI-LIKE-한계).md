# MySQL Functional Index (FBI) + LIKE 한계

- **카테고리**: #DBMS #MySQL #index
- **태그**: #MySQL #index #FBI #functional-index #LIKE #REVERSE #튜닝
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-FBI(Function Based Index) + LIKE 동작테스트 - BigDataTeam]]

## 1. 핵심 요약

`LIKE '%suffix'`(접두 %)는 인덱스를 못 타므로 **`REVERSE(col)` 함수 기반 인덱스(FBI)** 로 우회 시도. 하지만 테스트 결과 **FBI는 동치(=) 조건만 인덱스를 타고, `REVERSE(col) LIKE 'xxx%'` 조합은 인덱스를 타지 못하는 버그**([#101207](https://bugs.mysql.com/bug.php?id=101207))가 있어 현 시점 사용 불가. → **생성 칼럼(Generated Column)+인덱스**로 우회한다.

---

## 2. FBI 생성·테스트

```sql
CREATE INDEX idx_tid_rev ON t ((REVERSE(transaction_id)));

-- ❌ LIKE: 인덱스 미사용 (type=ALL, possible_keys=NULL)
EXPLAIN SELECT * FROM t WHERE REVERSE(transaction_id) LIKE REVERSE('0017') || '%';

-- ✅ 동치(=): 인덱스 사용 (type=ref, key=idx_tid_rev)
EXPLAIN SELECT * FROM t WHERE REVERSE(transaction_id) = '9100061215202';
```

> `||`를 CONCAT으로 쓰려면 `sql_mode`에 `PIPES_AS_CONCAT` 필요.

## 3. 결론·해결

| 조건 | FBI 인덱스 |
|------|-----------|
| `REVERSE(col) = '...'` | ✅ 사용 |
| `REVERSE(col) LIKE 'xxx%'` | ❌ 미사용 (버그 #101207, possible_keys=NULL) |

→ **명시적 생성 칼럼 + 인덱스 + 컬럼 직접 조회**로 우회.

## 4. 연관 개념

- [[2026-06-14-MS07_(MySQL-Generated-Column-Virtual-Stored-인덱스)]] — FBI 대체(권장)
- [[2026-06-14-MS08_(MySQL-Full-Text-Search-MATCH-AGAINST-ngram)]]
