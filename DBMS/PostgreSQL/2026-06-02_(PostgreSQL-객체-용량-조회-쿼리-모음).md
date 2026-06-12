# PostgreSQL 객체·용량 조회 쿼리 모음
- **카테고리**: #DBMS #PostgreSQL
- **태그**: #PostgreSQL #admin #조회쿼리 #tablespace #index #partition
- **작성일**: 2026-06-02
- **참조 원본**: [[2026-06-02-database size 조회쿼리]], [[2026-06-02-PostgreSQL 데이터베이스 용량 확인을 위한 SQL 쿼리]], [[2026-06-02-tablespace 정보 조회 쿼리]], [[2026-06-02-index와 table 이 사용하는 tablespace 조회쿼리]], [[2026-06-02-tabespace 를 사용하는 객체 조회]], [[2026-06-02-현재 데이터베이스의 인덱스 상세 정보 조회를 위한 SQL 쿼리]], [[2026-06-02-현재데이터베이스의 index 정보정보 조회]], [[2026-06-02-현재 데이터베이스의 모든  제약조건 확인]], [[2026-06-02-파티션 테이블 관계 조회 쿼리]], [[2026-06-02-파티션 테이블의 부모테이블과 자식테이블 조회]]

## 1. 핵심 요약
- PostgreSQL 운영에서 자주 필요한 객체 정보 및 용량 조회 쿼리 모음. DB 용량, Tablespace, Index, 파티션, 제약조건 조회 포함.

## 2. 상세 설명

### Database 용량 조회

```sql
-- 간단 조회
SELECT datname, pg_size_pretty(pg_database_size(datname)) AS size
FROM pg_database;

-- OID 포함 상세 조회
SELECT oid,
       pg_database.datname AS database_name,
       pg_size_pretty(pg_database_size(pg_database.datname)) AS size
FROM pg_database;
```

---

### Tablespace 조회

**Tablespace 경로 조회**:
```sql
SELECT
    CASE
        WHEN pg_tablespace_location(oid) = '' AND spcname = 'pg_default'
            THEN current_setting('data_directory') || '/base/'
        WHEN pg_tablespace_location(oid) = '' AND spcname = 'pg_global'
            THEN current_setting('data_directory') || '/global/'
        ELSE pg_tablespace_location(oid)
    END AS spclocation,
    spcname
FROM pg_tablespace;
```

**특정 Tablespace를 사용하는 객체 조회**:
```sql
SELECT c.relname AS object_name, c.relkind AS object_type,
       c.relfilenode AS file_number,
       COALESCE(pg_tablespace_location(c.reltablespace), 'default or database path') AS tablespace_location
FROM pg_class c
JOIN pg_database d ON d.datname = current_database()
WHERE (c.reltablespace = (SELECT oid FROM pg_tablespace WHERE spcname = 'sales_tbs'))
   OR (c.reltablespace = 0 AND d.dattablespace = (SELECT oid FROM pg_tablespace WHERE spcname = 'sales_tbs'));
```

**Index와 Table의 Tablespace 불일치 조회**:
```sql
-- 인덱스와 테이블이 다른 Tablespace를 사용하는 경우만 출력
SELECT i.relname AS index_name, tsi.spcname AS index_tbsp,
       t.relname AS table_name, tst.spcname AS table_tbsp
FROM pg_class t
JOIN pg_tablespace tst ON (t.reltablespace = tst.oid OR (t.reltablespace = 0 AND tst.spcname = 'pg_default'))
JOIN pg_index pgi ON pgi.indrelid = t.oid
JOIN pg_class i ON pgi.indexrelid = i.oid
JOIN pg_tablespace tsi ON (i.reltablespace = tsi.oid OR (i.reltablespace = 0 AND tsi.spcname = 'pg_default'))
WHERE i.relname NOT LIKE 'pg_toast%'
  AND i.reltablespace != t.reltablespace;
```

---

### Index 조회

**간단 Index 목록** (`\di+` 동일):
```sql
SELECT schemaname, tablename, indexname, indexdef
FROM pg_indexes
WHERE schemaname <> 'pg_catalog';
```

**상세 Index 정보** (크기·액세스 메서드 포함):
```sql
SELECT n.nspname AS "Schema", c.relname AS "Name",
    CASE c.relkind
        WHEN 'i' THEN 'index'
        WHEN 'I' THEN 'partitioned index'
    END AS "Type",
    pg_catalog.pg_get_userbyid(c.relowner) AS "Owner",
    c2.relname AS "Table",
    am.amname AS "Access method",
    pg_catalog.pg_size_pretty(pg_catalog.pg_table_size(c.oid)) AS "Size"
FROM pg_catalog.pg_class c
LEFT JOIN pg_catalog.pg_namespace n ON n.oid = c.relnamespace
LEFT JOIN pg_catalog.pg_am am ON am.oid = c.relam
LEFT JOIN pg_catalog.pg_index i ON i.indexrelid = c.oid
LEFT JOIN pg_catalog.pg_class c2 ON i.indrelid = c2.oid
WHERE c.relkind IN ('i','I')
  AND n.nspname <> 'pg_catalog'
  AND n.nspname !~ '^pg_toast'
  AND n.nspname <> 'information_schema'
  AND pg_catalog.pg_table_is_visible(c.oid)
ORDER BY 1, 2;
```

---

### 제약조건 조회

```sql
SELECT n.nspname AS table_schema, c.relname AS table_name,
    CASE
        WHEN con.conname IS NOT NULL THEN con.conname
        ELSE c.relname || '_' || a.attname || '_not_null'
    END AS constraint_name,
    CASE
        WHEN con.contype = 'p' THEN 'PRIMARY KEY'
        WHEN con.contype = 'u' THEN 'UNIQUE'
        WHEN con.contype = 'f' THEN 'FOREIGN KEY'
        WHEN con.contype = 'c' THEN 'CHECK'
        WHEN a.attnotnull        THEN 'NOT NULL'
    END AS constraint_type,
    a.attname AS column_name
FROM pg_class c
JOIN pg_namespace n   ON n.oid = c.relnamespace
JOIN pg_attribute a   ON a.attrelid = c.oid
LEFT JOIN pg_constraint con
       ON con.conrelid = c.oid
      AND (con.conkey @> array[a.attnum] OR con.conkey IS NULL)
WHERE n.nspname NOT IN ('pg_catalog', 'information_schema')
  AND c.relkind = 'r'
  AND (con.contype IS NOT NULL OR a.attnotnull)
  AND a.attname NOT IN ('cmax','cmin','ctid','tableoid','xmax','xmin')
ORDER BY table_schema, table_name, constraint_name;
```

---

### 파티션 테이블 조회

**파티션 자식 테이블 목록**:
```sql
SELECT inhrelid::regclass AS child
FROM pg_catalog.pg_inherits
WHERE inhparent = 'p_sales'::regclass;
```

**파티션 계층 구조 (부모·자식)**:
```sql
SELECT schema_name, table_name,
       COALESCE(parent_name, 'Root Partition') AS parent_name
FROM partition_hierarchy
ORDER BY schema_name, parent_name, table_name;
```

## 3. 연관 개념 (지식 연결)
- [[2026-06-02_(PostgreSQL-모니터링-쿼리-모음)]] — PostgreSQL 성능 모니터링 쿼리
- [[2026-06-02_(PostgreSQL-운영-유틸리티-모음)]] — psql 명령어 및 운영 유틸리티
