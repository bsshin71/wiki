# MySQL 대용량 Online 인덱스 추가 절차

- **카테고리**: #DBMS #MySQL #online-ddl
- **태그**: #MySQL #online-ddl #index #INPLACE #row-log-buffer #proxysql
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-대용량 테이블에 online 인덱스 추가절차 - BigDataTeam]]

## 1. 핵심 요약

대용량 테이블에 인덱스를 무중단 추가하는 실전 절차: ① `innodb_online_alter_log_max_size`를 1G 이상으로 증가 → ② `ALGORITHM=INPLACE, LOCK=NONE`으로 인덱스 추가 → ③ 작업 중 DML 정상 동작(단 row log buffer 초과 시 실패·rollback) → ④ 진행률·row_log_buf·단계별 시간 모니터링 → ⑤ ProxySQL에서 지연 확인.

---

## 2. 절차

```sql
-- 1) 로그 버퍼 증가
SET GLOBAL innodb_online_alter_log_max_size = 1073741824;  -- >1G

-- 2) 인덱스 추가 (시작/종료 시각 함께)
SELECT now();
ALTER TABLE T_DEAL_LM ADD INDEX idx_big_test (COMP_NO,ACC_NO,STATE,ORD_DATE),
  ALGORITHM=INPLACE, LOCK=NONE;
SELECT now();
```
- 3) 작업 중 I/U/D 정상 수행. DDL 중 DML은 row log buffer에 임시저장 후 반영 → **버퍼 초과 시 DDL 실패·rollback** → 버퍼 키우고 재시도.

## 3. 진행 모니터링

```sql
-- 진척률 + 종료 예상시간
SELECT esc.SQL_TEXT, estc.EVENT_NAME, estc.WORK_COMPLETED, estc.WORK_ESTIMATED,
  CONCAT(ROUND(@PROGRESS:=estc.WORK_COMPLETED/WORK_ESTIMATED*100,2),"%") AS 진척률,
  sys.format_time(@ELAPSED:=(estc.TIMER_END-estc.TIMER_START)) AS 실행시간,
  sys.format_time(@REMAIN:=FLOOR(@ELAPSED*(100/@PROGRESS)-@ELAPSED)) AS 남은시간
FROM performance_schema.events_statements_current esc
JOIN performance_schema.events_stages_current estc ON estc.NESTING_EVENT_ID=esc.EVENT_ID \G;

-- row log buffer 사용량 (high_alloc 으로 피크 확인)
SELECT event_name, sys.format_bytes(current_alloc), sys.format_bytes(high_alloc)
FROM sys.x$memory_global_by_current_bytes WHERE event_name='memory/innodb/row_log_buf'\G;
```

## 4. ProxySQL 지연 확인

```sql
Admin> SELECT hostgroup, schemaname, count_star, sum_time/1000000, max_time/1000000, substr(digest_text,1,50)
FROM stats_mysql_query_digest WHERE digest_text LIKE '%T_DEAL_LM%' ORDER BY max_time DESC LIMIT 10;
```

## 5. 연관 개념

- [[2026-06-14-MS12_(MySQL-Online-DDL-ALGORITHM-INSTANT-INPLACE-COPY)]]
- [[2026-06-14-MS11_(MySQL-대용량-테이블-DDL-주의사항-실패케이스)]]
- [[2026-06-14-MS10_(MySQL-Online-DDL-진행상황-모니터링)]]
