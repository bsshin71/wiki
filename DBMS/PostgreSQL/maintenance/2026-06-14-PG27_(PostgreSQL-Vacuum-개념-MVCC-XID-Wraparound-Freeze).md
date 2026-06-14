# PostgreSQL Vacuum 개념 (MVCC · XID Wraparound · Freeze)

- **카테고리**: #DBMS #PostgreSQL #구조
- **태그**: #PostgreSQL #vacuum #MVCC #XID #wraparound #freeze
- **작성일**: 2026-06-14
- **참조 원본**: [[postgresql-Vacuum-개념]] (raw/pdf2md/postgresql-Vacuum-개념.md, PDF 변환)

## 1. 핵심 요약

PostgreSQL은 MVCC로 변경 전/후 버전을 모두 보관하므로, **dead tuple 정리**와 **트랜잭션 ID(XID) Wraparound 방지(freeze)** 를 위해 Vacuum이 필수입니다.
XID는 4바이트(~43억)라 순환(wraparound)하며, Vacuum이 오래된 tuple을 **freeze**(영구 과거로 표시)하고 테이블 `relfrozenxid`를 갱신해 age를 낮춰 wraparound를 예방합니다.

---

## 2. MVCC (Multi-Version Concurrency Control)

- 쿼리 수행 시점의 데이터를 제공하는 기법.
- 현재 데이터(Current) + 변경 전 데이터(Before)를 함께 보관 → 동시성 보장.
- 부작용: **dead tuple(옛 버전) 누적** → Vacuum으로 정리 필요.

## 3. XID Wraparound

- 트랜잭션 ID(XID)는 **4바이트(~43억)** 로 관리.
- 43억 소진 시 1부터 재순환(wraparound) → **미래/과거 트랜잭션 판단 오류**로 데이터 가시성 붕괴 위험.
- 예: 1,000 TPS면 약 **49.7일**(43억 / (86,400초 × 1,000))에 소진.

## 4. Freeze로 Wraparound 방지

> Vacuum이 오래된 tuple을 **frozen**(항상 과거로 보이도록) 표시하고 테이블 age를 리셋.

| 항목 | 의미 |
|------|------|
| `pg_class.relfrozenxid` | 해당 테이블에 vacuum(freeze)을 수행했던 기준 XID |
| **age** | 현재 XID − relfrozenxid 로 계산한 "나이" |

### 동작 예 (autovacuum_freeze_max_age=1000, vacuum_freeze_min_age=100)
- XID=100에 테이블 A 생성 → XID=1100 시점에 age=1000 도달 → autovacuum 발생.
- freeze 후 `relfrozenxid`가 1000으로 갱신 → **age가 100으로 줄어듦**(테이블이 "젊어짐").

### age별 진행
1. 생성 → `autovacuum_freeze_max_age`만큼 증가 시 1차 autovacuum.
2. 이후 `freeze_max_age − vacuum_freeze_min_age` 주기로 계속 autovacuum.
3. age가 `vacuum_freeze_table_age` 도달 시 **테이블 전체 tuple을 읽어 freeze**.

> ⚠️ Anti-Wraparound Vacuum은 **autovacuum이 off여도 강제 실행**(데이터 보호).

## 5. Vacuum 모니터링

```sql
-- dead/live tuple 비율
SELECT relname, n_live_tup, n_dead_tup, n_dead_tup/(n_live_tup::float) AS ratio
FROM pg_stat_user_tables WHERE n_live_tup>0 AND n_dead_tup>1000 ORDER BY ratio DESC;

-- 마지막 (auto)vacuum 시각
SELECT relname, last_vacuum, last_autovacuum, last_analyze, last_autoanalyze
FROM pg_stat_user_tables ORDER BY relname;

-- 테이블 age 확인 (wraparound 임박 점검)
SELECT relname, age(relfrozenxid) FROM pg_class WHERE relkind='r' ORDER BY age(relfrozenxid) DESC;
```

## 6. 연관 개념

- [[2026-06-14-PG17_(PostgreSQL-AutoVacuum-설정-및-튜닝)]]
- [[2026-06-14-PG02_(PostgreSQL-아키텍처-및-특징)]]
- [[2026-06-14-PG21_(PostgreSQL-문제상황-Index-Bloating-Deadlock)]]
