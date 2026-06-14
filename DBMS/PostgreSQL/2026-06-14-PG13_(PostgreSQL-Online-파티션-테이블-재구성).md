# PostgreSQL Online 파티션 테이블 재구성

- **카테고리**: #DBMS #PostgreSQL #admin
- **태그**: #PostgreSQL #partition #online #reorg #lock #rename
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-online 파티션 테이블 schema 변경 - BigDataTeam]]

## 1. 핵심 요약

대용량 파티션 테이블 구조를 운영 중 변경하는 절차로, **신규 파티션 테이블 생성 → 과거 데이터 사전 이관 → 짧은 LOCK 구간에서 잔여 이관 + table rename switch**로 다운타임을 최소화합니다.
**INSERT만 발생하는 로그성 데이터**에 적합하며, rename 후 시퀀스 리셋·하위 파티션 권한 재부여가 필요합니다.

---

## 2. 적용 조건

- INSERT만 발생(UPDATE/DELETE 거의 없음).
- 로그성 데이터.

## 3. 신규 파티션 테이블 생성

```sql
CREATE TABLE public.new_useract_log (LIKE public.useract_log INCLUDING ALL)
  PARTITION BY RANGE (udate);
ALTER TABLE public.new_useract_log OWNER TO postgres;
GRANT INSERT,DELETE,UPDATE,SELECT ON public.new_useract_log TO bosloguser;

-- default + 월별 파티션
CREATE TABLE public.useract_log_pdefault PARTITION OF public.new_useract_log DEFAULT;
CREATE TABLE public.useract_log_p2025_08 PARTITION OF public.new_useract_log
  FOR VALUES FROM ('2025-08-01 00:00:00+00') TO ('2025-09-01 00:00:00+00');
-- ... 월별 반복
```

## 4. 과거 데이터 사전 이관 (online)

```sql
INSERT INTO public.new_useract_log OVERRIDING SYSTEM VALUE
  SELECT * FROM public.useract_log WHERE udate < '2025-12-02 00:00:00+00';
```

## 5. LOCK 구간: 잔여 이관 + rename switch

```sql
BEGIN;
LOCK TABLE public.useract_log IN ACCESS EXCLUSIVE MODE;   -- 쓰기 차단(짧게)
INSERT INTO public.new_useract_log OVERRIDING SYSTEM VALUE
  SELECT * FROM public.useract_log WHERE udate >= '2025-12-02 00:00:00+00';

ALTER TABLE useract_log     RENAME TO useract_log_old;     -- 기존 백업
ALTER TABLE new_useract_log RENAME TO useract_log;         -- 신규를 운영명으로

-- id 시퀀스 MAX+1 리셋
SELECT setval(pg_get_serial_sequence('useract_log','id'),
              COALESCE((SELECT MAX(id) FROM useract_log)+1, 1), false);
COMMIT;
```

## 6. ⚠️ rename 후 권한 재부여

> rename/truncate 시 table oid 변경으로 하위 파티션 권한이 사라질 수 있음.

```sql
-- 권한 확인
SELECT table_name, grantee, privilege_type FROM information_schema.table_privileges
WHERE table_schema='public' AND table_name LIKE 'useract_log%' AND grantee='bosloguser';

-- 재부여
GRANT ALL ON ALL TABLES IN SCHEMA public TO postgres;
GRANT INSERT,SELECT,UPDATE,DELETE ON ALL TABLES IN SCHEMA public TO bosloguser;
```

## 7. DO 블록으로 1년치 파티션 일괄 추가

```sql
DO $$
DECLARE m INT; start_date TIMESTAMPTZ; end_date TIMESTAMPTZ; part_name TEXT;
BEGIN
  FOR m IN 1..12 LOOP
    start_date := to_timestamp('2026-'||m||'-01','YYYY-MM-DD');
    end_date   := start_date + interval '1 month';
    part_name  := format('useract_log_p%4s_%s', 2026, to_char(m,'FM00'));
    EXECUTE format($sql$CREATE TABLE public.%I PARTITION OF public.useract_log
                        FOR VALUES FROM ('%s') TO ('%s');$sql$, part_name, start_date, end_date);
    EXECUTE format('GRANT UPDATE,SELECT,DELETE,INSERT ON public.%I TO bosloguser, loguser;', part_name);
  END LOOP;
END $$ LANGUAGE plpgsql;
```

## 8. 연관 개념

- [[2026-06-14-PG11_(PostgreSQL-pg_partman-파티션-자동관리)]]
- [[2026-06-14-PG07_(PostgreSQL-권한-관리-GRANT)]]
- [[2026-06-14-PG10_(PostgreSQL-시퀀스-관리)]]
