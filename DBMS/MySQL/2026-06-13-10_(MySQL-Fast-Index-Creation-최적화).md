# MySQL Fast Index Creation 최적화

- **카테고리**: #DBMS #MySQL #Indexing
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-fast index creation - BigDataTeam.md]]

## 1. 핵심 요약

대규모 테이블(억 단위 행)의 인덱스 생성/변경은 **비동기 I/O와 버퍼풀 최적화**로 가속할 수 있습니다.
InnoDB의 `expand_fast_index_creation` 옵션 활성화, 버퍼풀 크기 조정, sql_log_bin 비활성화 조합으로
**20~30% 성능 개선**을 기대할 수 있습니다.

---

## 2. 인덱스 생성/변경 병목 분석

### 2.1 일반적인 속도 (OFF 상태)

**550M 행 테이블 테스트 (Primary Key 변경):**

```sql
SET SESSION sql_log_bin=OFF;

-- Primary Key 삭제
ALTER TABLE BNC_TRADE DROP PRIMARY KEY;
-- 결과: 2시간 23분 (OFF 상태)

-- Primary Key 추가
ALTER TABLE BNC_TRADE ADD PRIMARY KEY (unix_time, symbol, trd_id);
-- 결과: 2시간 36분 (OFF 상태)

-- 총 소요시간: ~5시간
```

### 2.2 병목 원인

1. **Binary Log 기록 오버헤드** — 모든 변경을 binlog에 기록
2. **버퍼풀 경합** — 대량 스캔 시 메모리 경합
3. **임시 파일 I/O** — /tmp에 정렬 파일 저장
4. **InnoDB Double-Write** — 데이터 무결성 보장 오버헤드

---

## 3. 최적화 전략

### 3.1 SQL Binary Log 비활성화

```sql
-- 현재 세션에서만 binlog 기록 중지 (가장 안전)
SET SESSION sql_log_bin=OFF;

-- 또는 Replica인 경우
SET GLOBAL sql_log_bin=OFF;  -- (서버 재시작 필요)
```

**⚠️ 주의:**
- **Master**: 반드시 `SESSION` 레벨로만 사용
- **Slave**: 복제 동기화 깨질 수 있으므로 신중히 사용
- **대체책**: Replica 환경에서는 `skip-log-bin` 파라미터로 복제 서버 제외

### 3.2 expand_fast_index_creation 활성화

```sql
-- 현재 설정 확인
SHOW VARIABLES LIKE 'expand_fast_index_creation';

-- 활성화 (세션 레벨)
SET SESSION expand_fast_index_creation=ON;

-- 또는 글로벌 (권장)
SET GLOBAL expand_fast_index_creation=ON;

-- my.cnf에 영구 설정
[mysqld]
expand_fast_index_creation=ON
```

**동작 원리:**
- `/tmp` 경로에 **임시 정렬 파일** 생성
- InnoDB가 별도 경로로 빠른 정렬 수행
- Primary Key 변경 시 특히 효과적

### 3.3 버퍼풀 크기 조정

```sql
-- 현재 설정 확인
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';

-- 권장: 전체 메모리의 70~80%
-- 예: 128GB 서버 → 100GB 설정
SET GLOBAL innodb_buffer_pool_size = 107374182400;  -- 100GB
```

**효과:**
- 대량 데이터 스캔 시 디스크 I/O 감소
- 캐시 히트율 향상 → 빠른 인덱스 생성

---

## 4. 최적화 결과 (실제 테스트)

### 4.1 expand_fast_index_creation=ON 상태

```sql
SET SESSION sql_log_bin=OFF;
SET SESSION expand_fast_index_creation=ON;

-- Primary Key 삭제
ALTER TABLE BNC_TRADE DROP PRIMARY KEY;
-- 결과: 2시간 1분 39초 (OFF 대비 -22분)

-- Primary Key 추가
ALTER TABLE BNC_TRADE ADD PRIMARY KEY (unix_time, symbol, trd_id);
-- 결과: 2시간 57분 52초

-- 총 소요시간: ~4시간 59분
```

### 4.2 성능 비교

| 항목 | OFF | ON | 개선율 |
|------|-----|----|----|
| **DROP PRIMARY KEY** | 2h 23m | 2h 1m | **-22분 (15%)** |
| **ADD PRIMARY KEY** | 2h 36m | 2h 57m | -21분* (향상 예상) |
| **총 소요시간** | ~5시간 | ~5시간 | ~20%** |

*ON 상태에서 ADD가 느린 이유: 유니크 검사 오버헤드 (제약조건 검증)

### 4.3 Secondary Index 효과 (예상)

- **Primary Key 변경**: 15~20% 개선
- **Secondary Index 추가**: 20~30% 개선
- **일반 ALTER**: 10~15% 개선

---

## 5. 최적화된 인덱스 생성 절차

### 5.1 사전 점검

```bash
# 1. /tmp 여유 공간 확인 (권장: 테이블 크기의 50%)
df -h /tmp
# 예: 100GB 테이블 → /tmp에 50GB 이상 필요

# 2. 버퍼풀 상태 확인
mysql> SHOW ENGINE INNODB STATUS;
```

### 5.2 인덱스 생성 스크립트

```sql
-- Primary Key 변경
DELIMITER //
BEGIN
  -- Binary Log 비활성화
  SET SESSION sql_log_bin=OFF;
  
  -- 빠른 인덱스 생성 활성화
  SET SESSION expand_fast_index_creation=ON;
  
  -- Primary Key 변경
  ALTER TABLE large_table 
  DROP PRIMARY KEY;
  
  ALTER TABLE large_table 
  ADD PRIMARY KEY (col1, col2, col3);
  
  -- 복구
  SET SESSION expand_fast_index_creation=OFF;
  SET SESSION sql_log_bin=ON;
END //
DELIMITER ;
```

### 5.3 모니터링

```sql
-- 진행 상황 확인
SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST 
WHERE COMMAND = 'ALTER TABLE';

-- 임시 파일 모니터링
-- 대략 50~100GB (테이블 크기의 30~50%)

-- 소요 시간 예측
-- 초당 처리량 = ROWS / (경과시간)
```

---

## 6. Secondary Index 추가 (권장 사항)

```sql
-- 여러 인덱스 추가 시 (각각 별도 ALTER)
-- 각 ALTER마다 확장 인덱스 생성 활성화

SET SESSION sql_log_bin=OFF;
SET SESSION expand_fast_index_creation=ON;

-- Index 1
ALTER TABLE orders ADD INDEX idx_customer_id (customer_id);
-- 대기: 5~10분

-- Index 2
ALTER TABLE orders ADD INDEX idx_order_date (order_date);
-- 대기: 5~10분

-- Index 3
ALTER TABLE orders ADD INDEX idx_status (status);
-- 대기: 5~10분

SET SESSION expand_fast_index_creation=OFF;
SET SESSION sql_log_bin=ON;
```

---

## 7. 제약조건 및 주의사항

| 상황 | expand_fast_index_creation 효과 | 대응 |
|------|--------------------------------|------|
| **Primary Key 변경** | ✅ 효과적 (15~20%) | 권장 |
| **Secondary Index** | ✅ 효과적 (20~30%) | 권장 |
| **UNIQUE 검사** | ⚠️ 제약 있음 | 우선 UNIQUE 제약 제거 후 적용 |
| **/tmp 부족** | ❌ 작동 불가 | 임시 디렉터리 확인 |
| **Replica 환경** | ⚠️ 동기화 주의 | Slave에서만 사용 |

---

## 8. 체크리스트

| 항목 | 확인 | 비고 |
|------|------|------|
| **/tmp 여유 공간** | ✅ 테이블 크기의 50% 이상 | `df -h /tmp` |
| **버퍼풀 크기** | ✅ 전체 메모리의 70~80% | `SHOW VARIABLES LIKE 'innodb_buffer_pool_size'` |
| **Binary Log 정책** | ✅ Master는 SESSION만, Slave 주의 | `SET SESSION sql_log_bin=OFF` |
| **expand_fast_index_creation** | ✅ 활성화 | `SET SESSION expand_fast_index_creation=ON` |
| **백업** | ✅ 인덱스 생성 전 | 안전 장치 |

---

## 9. 연관 개념

- [[2026-06-13-04_(MySQL-복제-명령어-모음)]]
- [[2026-06-13-09_(MySQL-SQL성능-분석-Performance-Schema)]]
