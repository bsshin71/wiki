# MySQL Online DDL 진행상황 모니터링

- **카테고리**: #DBMS #MySQL #monitoring
- **태그**: #MySQL #monitoring #online-ddl #performance_schema #대용량작업 #row-log-buffer
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-online DDL 진행상황 보기 - BigDataTeam]]

## 1. 핵심 요약

Online DDL(`ALTER ... ALGORITHM=INPLACE, LOCK=NONE`)의 진척률은 **Performance Schema의 `stage/innodb/alter%` 인스트루먼트와 `events_stages_current`** 의 `WORK_COMPLETED/WORK_ESTIMATED`로 추정한다. 사전에 setup_instruments/consumers를 활성화해야 하며, **row log buffer(`innodb_online_alter_log_max_size`)** 가 한도를 넘으면 DDL이 실패하므로 대용량 작업 전 값을 키운다.

---

## 2. 모니터링 사전 설정

```sql
UPDATE performance_schema.setup_instruments SET enabled='YES'
  WHERE name LIKE 'stage/innodb/alter%';
UPDATE performance_schema.setup_consumers SET enabled='YES'
  WHERE name LIKE 'events_stage%';
```

## 3. 진척률·종료 예상시간 조회

```sql
SELECT esc.SQL_TEXT, estc.EVENT_NAME,
  estc.WORK_COMPLETED, estc.WORK_ESTIMATED,
  CONCAT(ROUND(estc.WORK_COMPLETED/WORK_ESTIMATED*100,2),"%") AS 진척률,
  sys.format_time(estc.TIMER_END - estc.TIMER_START) AS 실행시간
FROM performance_schema.events_statements_current esc
JOIN performance_schema.events_stages_current estc
  ON estc.NESTING_EVENT_ID = esc.EVENT_ID \G;
```
- 결과가 안 나오면 `metadata_locks`+`threads` 조인으로 thread 확인, `events_stages_history` 조회.
- 단계 예: `read PK and internal sort` → `merge sort` → `insert` → `log apply index/table`.

## 4. Row Log Buffer · 실패 방지

```sql
SHOW GLOBAL VARIABLES LIKE '%online%';   -- innodb_online_alter_log_max_size (기본 128M)
SET GLOBAL innodb_online_alter_log_max_size = 1073741824;  -- 1G로 증가
```
- DDL 중 발생한 DML이 row log buffer에 쌓이며, **버퍼 초과 시 DDL 실패**.
- (MariaDB) `Innodb_onlineddl_rowlog_pct_used`, `Innodb_onlineddl_pct_progress` status로 사용량·진행 확인.

## 5. 연관 개념

- [[2026-06-14-MS01_(MySQL-InnoDB-Adaptive-Hash-Index-AHI)]]
- [[2026-06-13-24_(MySQL-InnoDB-Lock-모니터링)]]
