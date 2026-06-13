# MySQL 쿼리 로깅 (General Log / Slow Log)

- **카테고리**: #DBMS #MySQL #admin
- **태그**: #admin #모니터링 #general_log #slow_query_log #로깅
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-쿼리 로깅하기 - BigDataTeam]]

## 1. 핵심 요약

**모든 쿼리는 general log, 느린 쿼리는 slow log**에 기록할 수 있으며, `log_output`으로 FILE/TABLE/둘 다 출력 위치를 정합니다.
slow log는 `long_query_time` 임계와 Percona 확장(`log_slow_rate_limit`, `slow_query_log_use_global_control` 등)으로 정교하게 제어합니다.

---

## 2. General Log

```sql
SHOW VARIABLES LIKE '%general%';       -- 현재 설정
SET GLOBAL general_log = 'ON';          -- 켜기 / 'OFF' 끄기
```

### 2.1 출력 위치 (log_output)
```sql
SHOW VARIABLES LIKE 'log_output';
SET GLOBAL LOG_OUTPUT = 'TABLE';        -- DB 테이블
SET GLOBAL LOG_OUTPUT = 'FILE';         -- 파일
SET GLOBAL LOG_OUTPUT = 'FILE,TABLE';   -- 동시
```

### 2.2 TABLE 조회 (argument는 BLOB → convert 필요)
```sql
SELECT *, CONVERT(argument USING utf8) FROM mysql.general_log LIMIT 10;
```

---

## 3. Slow Log

```sql
SHOW VARIABLES LIKE 'slow_query_%';
SET GLOBAL slow_query_log = 'ON';       -- 켜기 / 'OFF' 끄기
```

### 3.1 기준 시간
```sql
SHOW VARIABLES LIKE 'long_query_time';
SET GLOBAL long_query_time = 0;         -- 0 = 모든 쿼리 로깅
```

### 3.2 TABLE 조회 (sql_text는 BLOB → convert)
```sql
SELECT *, CONVERT(sql_text USING utf8) FROM mysql.slow_log LIMIT 10;
```

### 3.3 slow_log 테이블 truncate
```sql
SET GLOBAL slow_query_log = 'OFF';
TRUNCATE TABLE mysql.slow_log;
SET GLOBAL slow_query_log = 'ON';
```

---

## 4. Percona 확장 slow log 설정

| 변수 | 설명 |
|------|------|
| `log_slow_rate_limit` | =1 모든 느린 쿼리 로깅, =N(>1) N번째마다만 로깅 |
| `slow_query_log_use_global_control` | 런타임 변경을 현재 연결된 모든 세션에 적용 (`all` 또는 항목 지정). 기본은 신규 세션에만 적용 |
| `slow_query_log_always_write_time` | 지정 시간(예 1초) 초과 쿼리는 `log_slow_rate_limit` 무관 무조건 기록 |

---

## 5. 버전별 slow log 주의

> MySQL 8.4 기준: **초기 락 획득 시간은 실행 시간으로 계산되지 않음**.
> 세션1이 `c1=3` 행을 잠근 상태에서 세션2가 동일 행 update로 대기할 때, 대기(락 획득) 시간이 실행 시간에서 제외되어 slow log에 안 잡힐 수 있음.

- 참고: https://dev.mysql.com/doc/refman/8.4/en/slow-query-log.html

---

## 6. 연관 개념

- [[2026-06-13-05_(MySQL-Admin-쿼리-기초)]]
- [[2026-06-13-19_(MySQL-Audit-로깅)]]
- [[2026-06-13-09_(MySQL-SQL성능-분석-Performance-Schema)]]
