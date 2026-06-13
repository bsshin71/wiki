# MySQL Binary Log 활성화/비활성화

- **카테고리**: #DBMS #MySQL #BinLog
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-MySQL bin log 활성비활성 작업 - BigDataTeam.md]]

## 1. 핵심 요약

Binary Log는 모든 쓰기 작업을 기록하며 복제 및 Point-in-Time Recovery에 필수입니다.
데이터 변경 없이 쿼리만 실행하는 경우 또는 임시 작업 시 sql_log_bin=OFF로 비활성화할 수 있습니다.
⚠️ 레플리카 환경에서는 극도의 주의가 필요합니다.

---

## 2. Binary Log 확인

```sql
-- 활성화 상태
SHOW VARIABLES LIKE 'log_bin';

-- 현재 바이너리 로그 파일
SHOW MASTER STATUS;

-- 바이너리 로그 목록
SHOW BINARY LOGS;
```

---

## 3. 세션 단위 비활성화

```sql
-- 임시 비활성화 (현재 세션만)
SET SESSION sql_log_bin = OFF;

-- 쿼리 실행 (binlog에 기록되지 않음)
INSERT INTO table VALUES (...);
UPDATE table SET ... WHERE ...;

-- 재활성화
SET SESSION sql_log_bin = ON;
```

**⚠️ 주의**: Master에서는 SESSION 레벨만 사용

---

## 4. 글로벌 비활성화 (재시작 필요)

```ini
# my.cnf
[mysqld]
skip-log-bin

# 또는 로그 이름만 지정 (생성 안 함)
log-bin=OFF
```

**주의**: 전체 서버 재시작 필요

---

## 5. Replication 환경에서의 주의

| 환경 | SQL_LOG_BIN | 영향 |
|------|-----------|------|
| **Master** | OFF (SESSION) | 안전 (복제 감지 안 됨) |
| **Master** | OFF (GLOBAL) | ⚠️ 복제 불가능 - 절대 금지 |
| **Slave** | OFF | ⚠️ Slave 동기화 깨짐 |

---

## 6. 대안: 선택적 로깅

```sql
-- 특정 데이터베이스만 기록
SET GLOBAL binlog_do_db = 'important_db';
SET GLOBAL binlog_ignore_db = 'temp_db';

SHOW VARIABLES LIKE 'binlog_%';
```

---

## 7. 연관 개념

- [[2026-06-13-01_(MySQL-Binary-Log-Position-복제)]]
- [[2026-06-13-04_(MySQL-복제-명령어-모음)]]
