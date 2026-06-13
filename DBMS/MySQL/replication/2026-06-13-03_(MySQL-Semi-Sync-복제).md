# MySQL Semi-Synchronous 복제

- **카테고리**: #DBMS #MySQL #Replication
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-03. semi sync replication - BigDataTeam.md]], [[2026-06-12-semi replication - BigDataTeam]]

## 1. 핵심 요약

Semi-Synchronous Replication은 **Master가 Slave의 Relay Log 기록을 확인한 후 application에 OK를 반환**하는 방식입니다.
기본 비동기 복제보다 데이터 안전성이 높지만, Relay Log 기록 대기로 **성능 저하**가 발생합니다.
따라서 **데이터 무결성 > 성능** 환경에서만 권장되며, 플러그인 설치로 활성화합니다.

---

## 2. 복제 방식 비교

### 2.1 Asynchronous (기본값)

```
Master Commit → Binary Log 기록 → Application OK 반환
                                  ↓
                            (비동기) Slave Relay Log 기록
```

**특징**: 매우 빠르지만, Master 장애 시 Slave 미적용 데이터 손실 가능

### 2.2 Semi-Synchronous (권장)

```
Master Commit → Binary Log 기록 → Slave로 전송
                                  ↓
                           Slave Relay Log 기록
                                  ↓
                           Master에 ACK 반환
                                  ↓
                          Application OK 반환
```

**특징**: 느리지만 데이터 안전성 높음. Master-Slave 간 병렬 처리 가능 (3번, 4번, 5번, 7번 구간)

---

## 3. Semi-Sync 설정

### 3.1 Master 플러그인 설치

```sql
INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';

-- 활성화
SET GLOBAL rpl_semi_sync_master_enabled=1;

-- 확인
SHOW VARIABLES LIKE 'rpl_semi_sync_master%';
```

### 3.2 Slave 플러그인 설치

```sql
INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';

-- 활성화
SET GLOBAL rpl_semi_sync_slave_enabled=1;

-- I/O Thread 재시작 (필수)
STOP SLAVE IO_THREAD;
START SLAVE IO_THREAD;
```

### 3.3 영구 설정 (my.cnf)

```ini
# Master
[mysqld]
rpl_semi_sync_master_enabled=1
rpl_semi_sync_master_timeout=1000      # 1초 타임아웃

# Slave
[mysqld]
rpl_semi_sync_slave_enabled=1
```

---

## 4. 주요 시스템 변수

| 변수 | Master/Slave | 설명 |
|------|--------------|------|
| `rpl_semi_sync_master_enabled` | Master | Semi-Sync 활성화 (1/0) |
| `rpl_semi_sync_master_timeout` | Master | Slave ACK 대기 시간 (ms, 기본 10초) |
| `rpl_semi_sync_master_wait_for_slave_count` | Master | ACK 받을 Slave 수 (기본 1개) |
| `rpl_semi_sync_master_wait_no_slave` | Master | Slave 부족 시: ON=Semi-Sync 유지, OFF=비동기로 전환 |
| `rpl_semi_sync_master_wait_point` | Master | ACK 대기 시점: AFTER_SYNC(기본, 안전) or AFTER_COMMIT(빠름) |
| `rpl_semi_sync_slave_enabled` | Slave | Semi-Sync 활성화 (1/0) |

---

## 5. Semi-Sync 동작 상태 모니터링

### 5.1 Master 상태

```sql
SHOW GLOBAL STATUS LIKE 'Rpl_semi_sync_master%';

-- 주요 항목:
-- Rpl_semi_sync_master_status          → ON (정상)
-- Rpl_semi_sync_master_clients         → 1이상 (연결된 Slave 수)
-- Rpl_semi_sync_master_tx_waits        → ACK 대기한 트랜잭션 수
-- Rpl_semi_sync_master_tx_avg_wait_time → 평균 대기 시간 (ms)
-- Rpl_semi_sync_master_no_tx          → Timeout 발생 트랜잭션 수
```

### 5.2 Slave 상태

```sql
SHOW GLOBAL STATUS LIKE 'Rpl_semi_sync_slave_status';
-- 값: ON (정상 작동 중)
```

---

## 6. Timeout 발생 시 처리

### 6.1 Timeout 메커니즘

```
rpl_semi_sync_master_timeout 시간(기본 10초) 내 ACK 미수신
    ↓
비동기 복제로 자동 전환
    ↓
데이터 안전성 ↓, 성능 ↑
```

### 6.2 Timeout 조정

```sql
-- Timeout 1초로 단축 (네트워크 빠른 경우)
SET GLOBAL rpl_semi_sync_master_timeout=1000;

-- Timeout 30초로 연장 (네트워크 느린 경우)
SET GLOBAL rpl_semi_sync_master_timeout=30000;
```

---

## 7. Semi-Sync 사용 시 주의사항

### ⚠️ 성능 vs 안정성 트레이드오프

| 고려사항 | 권장값 |
|---------|-------|
| **High-Throughput DB** | Semi-Sync 비추천 |
| **금융/의료 데이터** | Semi-Sync 권장 |
| **Master-Slave 지연 > 100ms** | 성능 영향 큼 |
| **다중 Slave** | `wait_for_slave_count` 조정 필요 |

### 기본 점검 목록

```
[ ] Master와 Slave 네트워크 대역폭 충분한가?
[ ] Master의 트랜잭션 부하가 높지 않은가?
[ ] All-or-nothing 데이터 일관성이 정말 필요한가?
[ ] Slave 추가 시 rpl_semi_sync_slave_enabled 설정했는가?
```

---

## 8. Failover 시나리오

### 8.1 Master 장애 발생

```
Semi-Sync ON 상태에서 Master 장애 발생
    ↓
이미 Slave에 저장된 데이터 = 안전함
    ↓
Slave 승격 가능 (데이터 손실 최소)
```

### 8.2 Slave ACK 미응답

```
rpl_semi_sync_master_timeout 초과
    ↓
자동 비동기 복제로 전환
    ↓
성능 우선 처리
    ↓
나중에 Semi-Sync 재활성화 가능
```

---

## 9. 성능 최적화 팁

### 9.1 AFTER_SYNC vs AFTER_COMMIT

```sql
-- AFTER_SYNC (기본, 더 안전)
-- Binary Log 기록 후 즉시 ACK 대기 → 더 느림
SET GLOBAL rpl_semi_sync_master_wait_point='AFTER_SYNC';

-- AFTER_COMMIT (빠름, 약간 위험)
-- Storage에 커밋 후 ACK 대기 → 약간 빠름
SET GLOBAL rpl_semi_sync_master_wait_point='AFTER_COMMIT';
```

### 9.2 다중 Slave 설정

```sql
-- 3개 중 최소 2개 Slave ACK만 대기
SET GLOBAL rpl_semi_sync_master_wait_for_slave_count=2;

-- 응답 없는 Slave로 인한 성능 저하 경감
SET GLOBAL rpl_semi_sync_master_wait_no_slave=0;  -- 비동기로 전환
```

---

## 10. 모니터링 쿼리

```sql
-- Semi-Sync 성능 분석
SELECT 
  VARIABLE_NAME,
  VARIABLE_VALUE
FROM performance_schema.global_status
WHERE VARIABLE_NAME LIKE 'Rpl_semi_sync_master%'
ORDER BY VARIABLE_NAME;

-- Timeout 발생률 확인
SELECT 
  (Rpl_semi_sync_master_no_tx / (Rpl_semi_sync_master_yes_tx + Rpl_semi_sync_master_no_tx)) * 100 AS timeout_percentage
FROM performance_schema.global_status;
```

---

## 11. 권장 사용 환경

✅ **Semi-Sync 권장**:
- 금융 거래, 의료 데이터 등 데이터 무결성 중요
- Slave와의 네트워크 지연 < 50ms
- 트랜잭션 부하 낮음
- RPO (Recovery Point Objective) 0 필요

❌ **Semi-Sync 비추천**:
- 고처리량 OLTP 시스템
- 네트워크 불안정 (Timeout 빈번)
- 성능 > 안전성 우선
- 다중 Slave 관리 복잡

---

## 12. 전체 my.cnf 예시 (Semi-Sync + GTID 결합)

운영 환경에서는 Semi-Sync를 GTID 복제와 함께 구성합니다. `server-id`는 서버마다 달라야 합니다.

```ini
# ===== Master (server-id=1) =====
[mysqld]
# replication
log-bin=mysql-bin
log_slave_updates = slave_only
gtid_mode=ON
session_track_gtids=OWN_GTID
enforce_gtid_consistency=ON
server-id=1

rpl_semi_sync_master_enabled=1
rpl_semi_sync_master_timeout=1000   # 1s
rpl_semi_sync_slave_enabled=1       # 양방향 대비

binlog_format=row
binlog_do_db=DBBOS

# consistency (선택)
innodb_flush_log_at_trx_commit=1
sync_binlog=1

# ===== Slave =====
# server-id=2  (Master와 달라야 함), 나머지는 동일
```

---

## 13. 선택적 복제 필터링 (replicate-do / rewrite)

특정 DB·테이블만 복제하거나 DB명을 바꿔 복제할 수 있습니다.

```ini
# Slave 측 my.cnf — DBBOS → DB_EXEC 로 이름 바꿔 특정 테이블만 복제
replicate-rewrite-db="DBBOS->DB_EXEC"
replicate-do-db="DB_EXEC"
replicate-do-table="DB_EXEC.CUST_MM"
replicate-do-table="DB_EXEC.ACC_MM"
replicate-do-table="DB_EXEC.ORD_RULE_LM"
# ... 필요한 테이블만 나열
```

> 복제 계정 생성 / CHANGE MASTER / 상태 확인 명령은 [[2026-06-13-04_(MySQL-복제-명령어-모음)]] 참조.

---

## 14. 연관 개념

- [[2026-06-13-01_(MySQL-Binary-Log-Position-복제)]]
- [[2026-06-13-02_(MySQL-GTID-기반-복제)]]
- [[2026-06-13-04_(MySQL-복제-명령어-모음)]]
