# PostgreSQL pg_cron batch 스케줄링

- **카테고리**: #DBMS #PostgreSQL #admin
- **태그**: #PostgreSQL #pg_cron #batch #스케줄링 #cron
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-pg_cron 을 통한 batch 작업 - BigDataTeam]], [[2026-06-13-Postgres admin pg_cron - BigDataTeam]]

## 1. 핵심 요약

pg_cron은 DB 내부에서 crontab 형식으로 SQL을 스케줄 실행하는 확장으로, `shared_preload_libraries`에 등록 후 재시작해야 합니다.
`cron.schedule`/`cron.schedule_in_database`로 잡을 등록하고, `cron.job`/`cron.job_run_details`로 관리·이력 조회하며, **이력 테이블이 무한 적체되므로 정리 잡**을 함께 둡니다.

---

## 2. 환경 설정

```sql
ALTER SYSTEM SET shared_preload_libraries = 'pg_cron';
-- → systemctl restart postgresql → SHOW shared_preload_libraries; 확인
CREATE EXTENSION IF NOT EXISTS pg_cron;
```
```bash
journalctl -u postgresql -b --no-pager | egrep -i 'pg_cron|cron'   # 로딩 확인
```
> pg_cron은 unix domain 접속 불가 → **TCP 연결 필요**. `~/.pgpass`(600)에 `127.0.0.1:5432:db:user:pw` 등록.

## 3. 잡 등록

```sql
-- 기본 (pg_cron이 설치된 DB)
SELECT cron.schedule('test_minutely', '* * * * *', $$SELECT now()$$);

-- 다른 DB의 프로시저 호출
SELECT cron.schedule_in_database(
  'menu_events_load_every_min', '*/1 * * * *',
  'CALL public.load_menu_events();', 'useract');
```

## 4. 조회 / 변경 / 삭제

```sql
-- 잡 메타
SELECT jobid, jobname, schedule, command, database, active FROM cron.job;

-- 실행 이력
SELECT j.jobname, d.status, d.return_message, d.start_time, d.end_time
FROM cron.job_run_details d JOIN cron.job j ON d.jobid=j.jobid
ORDER BY d.start_time DESC LIMIT 10;

-- 변경 (schedule/command/database/active)
SELECT cron.alter_job(jobid := 1, schedule := '*/5 * * * *');
SELECT cron.alter_job(jobid := 2, active := false);     -- 임시 중단

-- 삭제
SELECT cron.unschedule(jobid) FROM cron.job WHERE jobname = 'menu_events_load_every_min';
```

## 5. ⚠️ 이력(job_run_details) 정리 (필수)

```sql
-- 성공 30일 / 실패 7일 차등 보존
SELECT cron.schedule('purge_pgcron_history_daily_tiered', '5 0 * * *', $$
  DELETE FROM cron.job_run_details WHERE status <> 'succeeded' AND end_time < now() - interval '7 days';
  DELETE FROM cron.job_run_details WHERE status =  'succeeded' AND end_time < now() - interval '30 days';
$$);

-- 공간 회수
SELECT cron.schedule('vacuum_pgcron_history_daily', '10 0 * * *',
  $$VACUUM (ANALYZE) cron.job_run_details;$$);
```

## 6. 연관 개념

- [[2026-06-14-PG11_(PostgreSQL-pg_partman-파티션-자동관리)]]
- [[2026-06-02_(PostgreSQL-운영-유틸리티-모음)]]
