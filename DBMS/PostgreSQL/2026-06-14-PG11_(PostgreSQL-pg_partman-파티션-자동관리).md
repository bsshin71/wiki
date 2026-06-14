# PostgreSQL pg_partman 파티션 자동 관리

- **카테고리**: #DBMS #PostgreSQL #admin
- **태그**: #PostgreSQL #partition #pg_partman #retention #자동화
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-pg_partman 을 통한 파티션 자동생성,삭제 관리 - BigDataTeam]]

## 1. 핵심 요약

pg_partman으로 `create_parent`에 control 컬럼·interval·premake를 지정해 **시간 기반 파티션을 자동 생성**하고, `part_config`의 retention으로 오래된 파티션을 자동 삭제합니다.
유지보수는 `run_maintenance`(보통 crontab/pg_cron으로 매시 실행)로 수행하며, **default 파티션에 데이터가 쌓이면 이후 파티션 추가가 실패**하므로 복구 절차가 필요합니다.

---

## 2. 설정

```sql
CREATE SCHEMA IF NOT EXISTS partman;
CREATE EXTENSION IF NOT EXISTS pg_partman SCHEMA partman;

-- 부모 등록 (일 단위, 미래 7개 미리 생성)
SELECT partman.create_parent(
  p_parent_table := 'public.useract_log',
  p_control      := 'udate',
  p_interval     := '1 day',
  p_premake      := 7,
  p_start_partition := NULL,      -- 기존 파티션 있으면 NULL 권장
  p_default_table   := false      -- 새 default 미생성
);

-- 보존 정책: 6개월 보존 + 미래 7일
UPDATE partman.part_config
   SET premake = 7, retention = '6 months'::interval,
       retention_keep_table = false, retention_keep_index = false
 WHERE parent_table = 'public.useract_log';

SELECT partman.run_maintenance('public.useract_log');   -- 1회 유지보수
```

## 3. 주기 실행 (crontab)

```bash
0 * * * * psql -d useract -c "SELECT partman.run_maintenance('public.useract_log');"
```
> pg_cron으로도 등록 가능 → [[2026-06-14-PG12_(PostgreSQL-pg_cron-batch-스케줄링)]].

## 4. ⚠️ default 파티션 적체로 인한 추가 실패 복구

> 파티션 미생성 시 데이터가 `_default`에 들어가고, 이후 run_maintenance가 실패한다.

```sql
-- 1) 임시 백업 테이블 + default의 해당 구간 row 이동
CREATE TABLE public.useract_log_backup (LIKE public.useract_log INCLUDING ALL);
INSERT INTO public.useract_log_backup OVERRIDING SYSTEM VALUE
  SELECT * FROM public.useract_log_default WHERE udate >= '2025-09-12 00:00:00+00' ON CONFLICT DO NOTHING;
DELETE FROM public.useract_log_default WHERE udate >= '2025-09-12 00:00:00+00';

-- 2) premake 늘려서 파티션 추가
UPDATE partman.part_config SET premake = 100 WHERE parent_table='public.useract_log';
SET client_min_messages TO NOTICE;
SELECT partman.run_maintenance('public.useract_log');

-- 3) 백업 row 복귀 → premake 원복(7)
INSERT INTO public.useract_log (...) OVERRIDING SYSTEM VALUE SELECT ... FROM public.useract_log_backup;
UPDATE partman.part_config SET premake = 7 WHERE parent_table='public.useract_log';
```

## 5. 관리 대상에서 제거

```sql
DELETE FROM partman.part_config WHERE parent_table = 'public.useract_log';
-- 기존 파티션은 유지, 자동 생성 대상에서만 제외
```

## 6. 연관 개념

- [[2026-06-14-PG13_(PostgreSQL-Online-파티션-테이블-재구성)]]
- [[2026-06-14-PG12_(PostgreSQL-pg_cron-batch-스케줄링)]]
