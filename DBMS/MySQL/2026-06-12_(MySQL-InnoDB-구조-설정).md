# MySQL InnoDB 구조 및 설정
- **카테고리**: #DBMS #MySQL
- **태그**: #MySQL #InnoDB #구조 #admin #redo-log #undo #partition #change-buffer
- **작성일**: 2026-06-12
- **참조 원본**: [[2026-06-12-리두로그 및 로그버퍼 - BigDataTeam]], [[2026-06-12-리두로그 아카이빙 - BigDataTeam]], [[2026-06-12-리두로그 활성화 및 비활성화 - BigDataTeam]], [[2026-06-12-체인지버퍼 - BigDataTeam]], [[2026-06-12-mysql undo 관리 - BigDataTeam]], [[2026-06-12-mysql partition 관리 - BigDataTeam]], [[2026-06-12-auto_increment 의 사용시 주의점 - BigDataTeam]], [[2026-06-12-MySQL timezone 설정 - BigDataTeam]], [[2026-06-12-mysql character set 설정 - BigDataTeam]], [[2026-06-12-MySQL bin log 활성비활성 작업 - BigDataTeam]]

## 1. 핵심 요약
- `innodb_flush_log_at_trx_commit=1`(기본)이 데이터 안전 최고. 성능 우선 시 `2`로 타협.
- Redo Log 크기는 8.0.31+부터 `innodb_redo_log_capacity`로 동적 조정 가능.
- `auto_increment`는 MySQL 재시작 후 최댓값으로 초기화될 수 있음 (8.0에서 개선됨).

## 2. 상세 설명

### Redo Log (리두 로그)

**innodb_flush_log_at_trx_commit 설정**:

| 값 | 동작 | 내구성 | 성능 |
|----|------|--------|------|
| 0 | 1초마다 write+sync | 낮음 (비정상 종료 시 1초치 손실 가능) | 빠름 |
| 1 | 매 commit마다 write+sync (기본) | 최고 | 느림 |
| 2 | 매 commit마다 write, sync는 1초마다 | 중간 (OS 크래시 시 1초치 손실 가능) | 중간 |

**Redo Log 크기 관련 파라미터**:
```sql
-- 현재 설정 확인
SHOW VARIABLES LIKE 'innodb_log%';
SHOW VARIABLES LIKE 'innodb_flush%';

-- 8.0.31+ 동적 조정
SET GLOBAL innodb_redo_log_capacity = 12 * 1024 * 1024 * 1024;  -- 12GB
```

**주요 파라미터**:
| 파라미터 | 설명 |
|----------|------|
| `innodb_log_file_size` | Redo log 파일 크기 |
| `innodb_log_files_in_group` | Redo log 파일 개수 |
| `innodb_log_buffer_size` | Log buffer pool 크기 (기본 16M) |
| `innodb_flush_method` | O_DIRECT 권장 (이중 버퍼링 방지) |

---

### Undo 관리

```ini
# my.cnf
innodb_undo_tablespaces      = 2
innodb_max_undo_log_size     = 536870912   # 512MB
innodb_undo_log_truncate     = ON          # 자동 truncate
innodb_rollback_segments     = 64
```

```sql
-- Undo 테이블스페이스 상태 확인
SELECT * FROM information_schema.INNODB_TABLESPACES
WHERE name LIKE 'undo%';

-- Undo 로그 크기 모니터링
SHOW STATUS LIKE 'trx_rseg_history_len';
```

---

### Change Buffer (체인지 버퍼)

- 보조 인덱스(Secondary Index) 변경 사항을 버퍼에 모아서 배치 처리
- Primary Key 인덱스에는 적용 안 됨
- SSD 환경에서는 오히려 비효율적일 수 있어 비활성화 권장

```sql
-- Change Buffer 설정
SHOW VARIABLES LIKE 'innodb_change_buffer%';
-- innodb_change_buffering: all(기본), none, inserts, deletes, changes, purges

-- 상태 확인 (SHOW ENGINE INNODB STATUS에서)
SHOW ENGINE INNODB STATUS \G;
-- "INSERT BUFFER AND ADAPTIVE HASH INDEX" 섹션 확인
```

---

### 파티션 관리

```sql
-- 파티션 테이블 생성 예시
CREATE TABLE p_sales (
    id INT NOT NULL,
    sale_date DATE NOT NULL,
    amount DECIMAL(10,2)
) PARTITION BY RANGE (YEAR(sale_date)) (
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- 파티션 추가
ALTER TABLE p_sales ADD PARTITION (PARTITION p2025 VALUES LESS THAN (2026));

-- 파티션 삭제 (데이터 포함)
ALTER TABLE p_sales DROP PARTITION p2022;

-- 파티션 정보 확인
SELECT * FROM information_schema.PARTITIONS
WHERE TABLE_SCHEMA='db명' AND TABLE_NAME='p_sales';
```

---

### auto_increment 주의사항

```sql
-- 현재 auto_increment 값 및 최대값 확인
SELECT * FROM sys.schema_auto_increment_columns;

-- 사용률 확인
SELECT table_schema, table_name, column_name,
       auto_increment AS current_value, max_value,
       ROUND(auto_increment_ratio * 100, 2) AS usage_ratio
FROM sys.schema_auto_increment_columns;
```

> ⚠️ **MySQL 5.7 이하**: 서버 재시작 시 `MAX(id)+1`로 초기화됨 (삭제된 행의 ID 재사용 가능)
> **MySQL 8.0**: `redo log`에 auto_increment 값을 저장하여 재시작 후에도 유지

---

### Timezone 설정

```sql
-- 현재 timezone 확인
SELECT @@global.time_zone, @@session.time_zone;
SHOW VARIABLES LIKE 'time_zone';

-- 세션 timezone 변경
SET time_zone = 'Asia/Seoul';

-- 글로벌 설정 (my.cnf)
-- default-time-zone = '+09:00'   # 또는 'Asia/Seoul'
```

> ⚠️ `timezone_name` 사용 시 OS timezone 파일 필요. `+09:00` 형식이 더 안전.

---

### Character Set 설정

```sql
-- 현재 character set 확인
SHOW VARIABLES LIKE 'character%';
SHOW VARIABLES LIKE 'collation%';

-- DB 생성 시 명시
CREATE DATABASE mydb CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;

-- 테이블 변경
ALTER TABLE t1 CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
```

```ini
# my.cnf 권장 설정
character-set-server      = utf8mb4
character-set-filesystem  = utf8mb4
collation-server          = utf8mb4_0900_ai_ci
skip-character-set-client-handshake
```

---

### Binary Log 관리

```sql
-- Binary Log 상태 확인
SHOW BINARY LOGS;
SHOW MASTER STATUS;

-- 특정 시점까지 Binary Log 삭제
PURGE BINARY LOGS BEFORE '2026-06-01 00:00:00';
PURGE BINARY LOGS TO 'mysql-bin.000050';

-- Binary Log 비활성화 (세션, 주의)
SET SESSION sql_log_bin = 0;  -- 해당 세션만 적용
SET SESSION sql_log_bin = 1;  -- 복원

-- 만료 설정 (my.cnf)
-- binlog_expire_logs_seconds = 1209600  # 14일
```

## 3. 연관 개념 (지식 연결)
- [[2026-06-12_(MySQL-Percona-설치-가이드)]] — my.cnf 전체 설정 (InnoDB 파라미터 포함)
- [[2026-06-12_(MySQL-XtraBackup-백업복구-가이드)]] — Redo Log 관련 복구 원리
- [[2026-06-12_(MySQL-Performance-Schema-활용)]] — InnoDB 성능 모니터링
