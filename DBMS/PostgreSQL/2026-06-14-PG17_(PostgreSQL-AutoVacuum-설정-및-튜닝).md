# PostgreSQL AutoVacuum 설정 및 튜닝

- **카테고리**: #DBMS #PostgreSQL #쿼리튜닝
- **태그**: #PostgreSQL #autovacuum #vacuum #튜닝 #dead_tuple #wraparound
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-AutoVacuum 튜닝 작업 - BigDataTeam]], [[2026-06-14-AutoVacuum 설정 변수 - BigDataTeam]]

## 1. 핵심 요약

AutoVacuum은 dead tuple 누적치가 **`threshold + scale_factor × 행수`** 를 넘거나 테이블 age가 **`autovacuum_freeze_max_age`(2억)** 를 넘으면(Anti-Wraparound, off여도 강제 실행) 동작합니다.
수동 vacuum보다 **설정 튜닝으로 AutoVacuum이 원활히 돌게** 하는 것이 운영 목표이며, scale_factor·cost_limit·naptime·max_workers를 상황에 맞게 조정합니다.

---

## 2. 주요 설정 변수 (default)

| 변수 | default | 설명 |
|------|---------|------|
| `autovacuum` | on | 활성화(track_counts도 필요) |
| `autovacuum_max_workers` | 3 | 동시 worker 수 |
| `autovacuum_naptime` | 1min | 실행 사이 최소 지연 |
| `autovacuum_vacuum_threshold` | 50 | VACUUM 트리거 최소 변경 tuple |
| `autovacuum_vacuum_scale_factor` | 0.2 | 테이블 크기 대비 비율(20%) |
| `autovacuum_analyze_scale_factor` | 0.1 | ANALYZE 비율(10%) |
| `autovacuum_freeze_max_age` | 2억 | Anti-Wraparound 강제 VACUUM age |
| `autovacuum_vacuum_cost_delay` | 2ms | 비용 지연 |
| `autovacuum_vacuum_cost_limit` | -1 | -1=vacuum_cost_limit(200) 사용 |

## 3. AutoVacuum 실행 조건

### 3.1 Dead Tuple 임계치
```
vacuum threshold = autovacuum_vacuum_threshold + autovacuum_vacuum_scale_factor × tuple수
```
> 기본: dead tuple이 **전체 행의 20% + 50개** 초과 시 실행.

### 3.2 age 임계치 (Anti-Wraparound)
- 테이블 age > `autovacuum_freeze_max_age`(2억) → 강제 실행(AutoVacuum off여도).
- `vacuum_freeze_table_age` 초과 시 일반 vacuum 호출에 freeze 동반 → 동시 wraparound vacuum 분산.

## 4. 모니터링

```sql
-- dead/live tuple 비율
SELECT relname, n_live_tup, n_dead_tup, n_dead_tup/(n_live_tup::float) AS ratio
FROM pg_stat_user_tables WHERE n_live_tup>0 AND n_dead_tup>1000 ORDER BY ratio DESC;

-- 마지막 (auto)vacuum 시각
SELECT relname, last_vacuum, last_autovacuum, last_analyze, last_autoanalyze
FROM pg_stat_user_tables;
```

## 5. 튜닝 가이드

### 5.1 너무 드물게 → scale_factor 하향
```sql
-- 큰 테이블 부하 균형: 10% + 50으로
-- (전역) autovacuum_vacuum_scale_factor = 0.1
-- (특정 테이블만)
ALTER TABLE tb_test SET (autovacuum_vacuum_scale_factor = 0.01, autovacuum_vacuum_threshold = 0);
```

### 5.2 너무 느림 → credit/cost·worker 조정
> credit이 소진되면 vacuum 종료 → 충분한 credit 필요.
> page_hit=1, page_miss=10, page_dirty=20 credit 소비.

- 권장: `autovacuum_vacuum_cost_limit = 1000`(또는 200×max_workers/3), `cost_delay = 5ms`
- `autovacuum_naptime = 5s`(기본 1분 → 단축)
- `autovacuum_max_workers`: CPU N cores/2~4 (과설정 주의, 파티션 테이블은 파티션별 worker)
- `max_parallel_maintenance_workers`: 기본 2 → 느리면 4~8
- `autovacuum_work_mem`(maintenance_work_mem): 주기당 수집 dead tuple 수 결정(메모리 과설정 주의)

## 6. 연관 개념

- [[2026-06-14-PG02_(PostgreSQL-아키텍처-및-특징)]]
- [[2026-06-02_(PostgreSQL-운영-유틸리티-모음)]]
