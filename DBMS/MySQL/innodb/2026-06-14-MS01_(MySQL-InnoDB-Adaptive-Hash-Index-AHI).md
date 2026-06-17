# MySQL InnoDB Adaptive Hash Index (AHI)

- **카테고리**: #DBMS #MySQL #innodb
- **태그**: #MySQL #innodb #AHI #hash-index #buffer-pool #튜닝
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-AHI(Adaptive Hash Index) - BigDataTeam]]

## 1. 핵심 요약

AHI는 InnoDB가 **자주 조회되는 키를 자동으로 해시 인덱스화**하여 B-Tree를 타지 않고 O(1)로 데이터에 접근하는 기능. Buffer Pool의 1/64로 시작하며 사용자는 **on/off만** 가능(대상 선정 불가). **동등 조회(=)** 에 유리하고 **범위 조회·자주 갱신되는 테이블에는 오히려 성능 저하**가 될 수 있어 워크로드별로 조정한다.

---

## 2. 개념·동작

- InnoDB가 쿼리 패턴(동일 키 반복 조회)을 분석 → 해시 테이블 자동 생성, 미사용 시 자동 삭제.
- Buffer Pool에 저장되어 메모리를 소비.

## 3. 장단점

| 장점 | 단점 |
|------|------|
| 동등 키 조회 빠름(O(1)) | 범위 조회(>=, <=, BETWEEN, LIKE)에 미사용 |
| B-Tree보다 낮은 검색 비용 | 메모리 사용 증가 |
| 자동 관리(생성 불필요) | 잦은 변경 테이블에서 동적 갱신 부담 → 성능 저하 가능 |

## 4. 설정·모니터링

```sql
SET GLOBAL innodb_adaptive_hash_index = ON;   -- / OFF
SHOW ENGINE INNODB STATUS;   -- "INSERT BUFFER AND ADAPTIVE HASH INDEX" 섹션
-- 메모리 사용량
SELECT event_name, CURRENT_NUMBER_OF_BYTES_USED
FROM performance_schema.memory_summary_global_by_event_name
WHERE EVENT_NAME='memory/innodb/adaptive hash index';
```

> ⚠️ 오래 운영된 테이블을 운영 중 DROP하면 AHI 재구성으로 성능 영향 가능 → 주의.

## 5. 언제 끄나

- 자주 갱신(INSERT/UPDATE/DELETE)되는 테이블 / 범위 검색 多 / 메모리 절약 필요 시.

## 6. 연관 개념

- [[2026-06-14-MS02_(MySQL-InnoDB-Storage-Engine-아키텍처)]] — AHI는 In-Memory 구조의 일부
- [[2026-06-13-30_(MySQL-Change-Buffer)]]
