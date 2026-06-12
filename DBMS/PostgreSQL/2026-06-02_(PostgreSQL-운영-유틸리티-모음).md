# PostgreSQL 운영 유틸리티 모음
- **카테고리**: #DBMS #PostgreSQL
- **태그**: #PostgreSQL #admin #psql #autovacuum #sample-data
- **작성일**: 2026-06-02
- **참조 원본**: [[2026-06-02-psql  주요 명령어]], [[2026-06-02-시스템설정(Parameter) 조회 및 변경]], [[2026-06-02-테이블에 설정된 auto vacuum 설정 확인]], [[2026-06-02-테이블에 대량의 랜덤 데이터를 삽입하는 쿼리]], [[2026-06-02-날짜 생성 및 판매 데이터 삽입을 위한 PLpgSQL 스크립트]]

## 1. 핵심 요약
- PostgreSQL 일상 운영에 필요한 psql 메타 명령어, 파라미터 조회/변경, Auto Vacuum 확인, 테스트 데이터 생성 스크립트 모음.

## 2. 상세 설명

### psql 주요 메타 명령어

| 명령어 | 설명 |
|--------|------|
| `\l` | 데이터베이스 목록 |
| `\dn` / `\dn *` | 스키마 목록 (시스템 제외 / 모두) |
| `\dt` | 사용자 테이블 목록 |
| `\dt *.*` | 모든 스키마 테이블 목록 |
| `\dt+` | 테이블 목록 + 크기·설명 |
| `\dv` | 사용자 뷰 목록 |
| `\di` / `\di+` | 인덱스 목록 / 상세 |
| `\d 테이블명` | 테이블 구조 확인 |
| `\d+ 테이블명` | 테이블 구조 + 스토리지 파라미터 |
| `\du` / `\du+` | 역할(Role) 목록 |
| `\dS` | 시스템 카탈로그 테이블 목록 |
| `\show 파라미터명` | 특정 설정값 확인 |
| `\show all` | 모든 설정값 확인 |

**테이블·뷰 SQL 쿼리 조회**:
```sql
-- 사용자 테이블 목록
SELECT schemaname, tablename, tableowner
FROM pg_catalog.pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema');

-- 사용자 뷰 목록
SELECT schemaname, viewname, viewowner
FROM pg_views
WHERE schemaname NOT IN ('pg_catalog', 'information_schema');

-- 현재 접속 유저 확인
SELECT current_user, session_user;
```

---

### 시스템 파라미터 조회 및 변경

**조회 방법**:
```sql
-- 특정 파라미터
SHOW max_connections;
SHOW shared_buffers;

-- pg_settings 뷰 (단위·재시작 여부 포함)
SELECT name, setting, unit, context, short_desc
FROM pg_settings
WHERE name = 'max_connections';
-- context = 'postmaster': 서버 재시작 필요

-- 설정 파일 경로 확인
SHOW config_file;
```

**자주 확인하는 파라미터**:

| 파라미터 | 의미 |
|----------|------|
| `port` | DB 포트 번호 |
| `max_connections` | 최대 동시 접속 수 |
| `shared_buffers` | 공유 메모리 크기 |
| `data_directory` | 데이터 저장 경로 |
| `log_directory` | 로그 저장 경로 |
| `work_mem` | 쿼리별 메모리 한도 |

**동적 변경 (재시작 불필요 파라미터)**:
```sql
ALTER SYSTEM SET idle_in_transaction_session_timeout = '5min';
SELECT pg_reload_conf();  -- 설정 반영
```

---

### Auto Vacuum 설정 확인

**테이블별 Auto Vacuum 개별 설정 확인**:
```sql
SELECT relname, reloptions
FROM pg_class
WHERE relname = '테이블명';
-- reloptions가 NULL이면 시스템 기본값 적용 중
```

**테이블 Vacuum 상태 및 Dead Tuple 확인**:
```sql
SELECT schemaname, relname,
       n_dead_tup,          -- Dead Tuple 수
       n_live_tup,          -- Active Tuple 수
       last_autovacuum,     -- 마지막 AutoVacuum 시간
       last_autoanalyze     -- 마지막 AutoAnalyze 시간
FROM pg_stat_user_tables
WHERE relname = '테이블명';
```

**시스템 전체 AutoVacuum 설정 확인**:
```sql
SELECT name, setting, unit, short_desc
FROM pg_settings
WHERE name LIKE 'autovacuum%';
```

> 💡 업데이트·삭제가 빈번한 대용량 테이블은 `autovacuum_vacuum_scale_factor`를 낮게 설정하여 더 자주 실행되도록 권장.

---

### 테스트 데이터 생성

**대량 랜덤 데이터 삽입 (generate_series 활용)**:
```sql
INSERT INTO p_sales (product_id, total_quantity, sale_date)
SELECT
    (random() * 100)::int + 1 AS product_id,
    (random() * 1000)::int + 1 AS total_quantity,
    generate_series('2023-01-01'::date, '2023-12-31'::date, '1 day'::interval) AS sale_date
FROM generate_series(1, 10000);
```

**PL/pgSQL 루프를 사용한 대량 데이터 삽입** (1천만 건):
```sql
DO $$
DECLARE
    i INT;
    generated_date DATE;
BEGIN
    FOR i IN 1..10000000 LOOP
        generated_date := DATE '2023-01-01' + (RANDOM() * 364)::INT;
        INSERT INTO new_sales (order_date, product_category, customer_id, quantity, price)
        VALUES (
            generated_date,
            CASE (i % 3)
                WHEN 0 THEN 'Electronics'
                WHEN 1 THEN 'Clothing'
                ELSE 'Home Appliances'
            END,
            (RANDOM() * 1000)::INT,
            (RANDOM() * 10 + 1)::INT,
            ROUND(RANDOM() * 100)::NUMERIC
        );
    END LOOP;
END $$;
```

## 3. 연관 개념 (지식 연결)
- [[2026-06-02_(PostgreSQL-모니터링-쿼리-모음)]] — 모니터링 쿼리 (pg_stat_activity 등)
- [[2026-06-02_(PostgreSQL-객체-용량-조회-쿼리-모음)]] — 객체 조회 쿼리
