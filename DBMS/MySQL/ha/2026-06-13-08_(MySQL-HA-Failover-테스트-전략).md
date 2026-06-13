# MySQL HA Failover 테스트 전략

- **카테고리**: #DBMS #MySQL #HA
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-BOS DB HA(failover) 테스트 - BigDataTeam.md]]

## 1. 핵심 요약

HA(High Availability) Failover 테스트는 **Master 장애 시 자동 Slave 승격**이 정상 작동하는지 검증하는 프로세스입니다.
ProxySQL(로드밸런싱) → MHA(자동 장애감지/승격) → PMM(모니터링) 구성에서 각 계층의 역할을 테스트하고,
클라이언트와 서버 양쪽에서 **데이터 무결성과 서비스 가용성**을 확인합니다.

---

## 2. HA 아키텍처

### 2.1 구성 요소

```
┌──────────────┐
│  Client App  │
└──────────────┘
       │
       ▼
┌──────────────────┐
│   ProxySQL       │  ← 연결 라우팅 & 자동 failover
│ (Proxy Layer)    │
└──────────────────┘
       │
   ┌───┴───┐
   ▼       ▼
┌─────┐ ┌─────┐
│ M   │ │ S1  │  ← MySQL Replication
│(M) │ │(S1) │
└─────┘ └─────┘
   △       │
   │       ▼
   │    ┌─────┐
   │    │ S2  │
   │    │(S2) │
   │    └─────┘
   │
   ▼
┌──────────────┐
│ MHA Manager  │  ← 자동 장애감지 & 승격
│ (Ha Mgmt)    │
└──────────────┘
   │
   ▼
┌──────────────┐
│  PMM Server  │  ← 모니터링 & 알람
│ (Monitoring) │
└──────────────┘
   │
   ▼
┌──────────────┐
│ Alert Server │  ← 알람 전송
│ (Alerting)   │
└──────────────┘
```

### 2.2 각 계층의 역할

| 계층 | 구성 요소 | 기능 |
|------|---------|------|
| **Application** | Client App | 비즈니스 로직 실행 |
| **Connection Pool** | ProxySQL | Master 장애 감지 후 자동 Slave 라우팅 |
| **Database** | MySQL M/S | 마스터-슬레이브 복제 |
| **Orchestration** | MHA Manager | Master 장애 감지 & 자동 Slave 승격 |
| **Monitoring** | PMM Server | 실시간 DB 상태 모니터링 |
| **Alerting** | Alert Server | 장애 알림 발송 |

---

## 3. Failover 테스트 범위

### 3.1 Client Side (애플리케이션 레벨)

**검증 항목:**

1. **ProxySQL 자동 감지**
   - Master 장애 후 ProxySQL이 상태 변화를 감지하는가?
   - Failover 완료 후 자동으로 Slave로 연결 전환하는가?

2. **연결 복구**
   - 기존 연결이 끊어졌을 때 애플리케이션이 재연결 로직을 수행하는가?
   - 재연결 후 정상 작동하는가?

3. **데이터 무결성**
   - Failover 중 실행된 쿼리가 손실되지 않는가?
   - 이미 커밋된 데이터만 반영되는가?

### 3.2 Server Side (데이터베이스 레벨)

**검증 항목:**

1. **MHA 자동 승격**
   - MHA가 Master 장애를 정확히 감지하는가?
   - Slave 중 가장 최신 복제 상태인 Slave를 Master로 승격하는가?

2. **데이터 동기화**
   - 신규 Master 선정 후 다른 Slave들이 올바르게 복제 재설정되는가?
   - 데이터 불일치가 없는가?

3. **오류 감지**
   - Failover 후 DB 무결성 검증 쿼리 통과하는가?
   - 복제 지연이 정상 범위 내인가?

### 3.3 Monitoring & Alerting

**검증 항목:**

1. **PMM 장애 감지**
   - PMM이 Master 다운을 감지하는가?
   - 감지 시간은 예상 범위 내인가?

2. **알람 발송**
   - MHA 이벤트 알람이 정상 발송되는가?
   - PMM 알람이 정상 발송되는가?
   - 알람 그룹 및 채널 설정이 정확한가?

---

## 4. 테스트 프로그램 설계

### 4.1 Simple Insert 테스트

**목적:** 대량 트랜잭션 중 Failover 발생 시 데이터 손실 여부 검증

```python
import mysql.connector
from mysql.connector import Error

def simple_insert_test():
    for i in range(1, 10000001):  # 1000만 건
        success = False
        for retry in range(1, 11):  # 최대 10회 재시도
            try:
                insert_data(i)
                success = True
                break
            except Error as e:
                if retry < 10:
                    reconnect_db()  # 자동 재연결
                else:
                    log_error(f"Failed after 10 retries: {e}")
                    raise
        if not success:
            break

def insert_data(seq_num):
    cursor.execute(
        "INSERT INTO test_table (id, data) VALUES (%s, %s)",
        (seq_num, f"data_{seq_num}")
    )
    connection.commit()
```

**검증 방법:**
```sql
-- Failover 후 데이터 확인
SELECT COUNT(*) FROM test_table;
SELECT MAX(id) FROM test_table;

-- 누락된 ID 확인
SELECT id FROM (
  SELECT 1 AS id UNION ALL
  SELECT id FROM test_table
) t1
LEFT JOIN test_table t2 ON t1.id = t2.id
WHERE t2.id IS NULL
LIMIT 10;
```

### 4.2 주문 부하 테스트

**목적:** 복잡한 업무 로직이 Failover 후에도 정상 작동하는지 검증

```python
def order_load_test():
    """
    복잡한 주문 처리 로직
    1. 주문 생성
    2. 상품 재고 차감
    3. 결제 처리
    4. 배송 준비
    """
    for order_id in range(1, 100001):  # 10만 건
        try:
            with connection.cursor() as cursor:
                # Step 1: 주문 생성
                cursor.execute(
                    "INSERT INTO orders (order_id, customer_id, total_amount) "
                    "VALUES (%s, %s, %s)",
                    (order_id, f"cust_{order_id % 1000}", 50000)
                )
                
                # Step 2: 재고 차감
                cursor.execute(
                    "UPDATE products SET stock = stock - 1 "
                    "WHERE product_id = %s",
                    (order_id % 100,)
                )
                
                # Step 3: 결제
                cursor.execute(
                    "INSERT INTO payments (order_id, amount, status) "
                    "VALUES (%s, %s, %s)",
                    (order_id, 50000, "COMPLETED")
                )
                
                # Step 4: 배송 준비
                cursor.execute(
                    "INSERT INTO shipments (order_id, status) "
                    "VALUES (%s, %s)",
                    (order_id, "READY")
                )
                
                connection.commit()
                
        except Error as e:
            connection.rollback()
            reconnect_db()  # Failover 자동 재연결
            log_error(f"Order {order_id} failed: {e}")
```

**검증 방법:**
```sql
-- 주문 데이터 일관성 확인
SELECT o.order_id, 
       COUNT(p.payment_id) AS payment_count,
       COUNT(s.shipment_id) AS shipment_count
FROM orders o
LEFT JOIN payments p ON o.order_id = p.order_id
LEFT JOIN shipments s ON o.order_id = s.order_id
GROUP BY o.order_id
HAVING COUNT(p.payment_id) != 1 OR COUNT(s.shipment_id) != 1
LIMIT 10;
```

---

## 5. Failover 시뮬레이션

### 5.1 Master 강제 종료

```bash
# Master 서버에서
kill -9 $(pgrep mysqld)  # 강제 종료

# 또는 MySQL에서
SHUTDOWN IMMEDIATE;
```

### 5.2 MHA 로그 확인

```bash
# MHA Manager 로그
tail -f /var/log/mha4mysql-manager/manager.log

# 예상 출력:
# 2026-06-13 10:30:45 [info] Master Host is down!
# 2026-06-13 10:30:46 [info] Selecting new master...
# 2026-06-13 10:30:47 [info] New master is Slave01
# 2026-06-13 10:30:50 [info] Failover completed successfully
```

### 5.3 ProxySQL 상태 확인

```bash
# ProxySQL admin 접속
mysql -h 127.0.0.1 -P 6032 -u admin -p

SHOW HOSTGROUPS;
# 예상: Master → Slave로 라우팅 변경

SELECT * FROM stats_mysql_connection_pool;
# 예상: 새로운 Master(구 Slave)로 연결 수 증가
```

---

## 6. 테스트 체크리스트

| # | 검증 항목 | 기대값 | 실제값 | 통과 |
|----|---------|-------|-------|------|
| 1 | Master 장애 감지 시간 | < 10초 | | ✅/❌ |
| 2 | Slave 승격 시간 | < 30초 | | ✅/❌ |
| 3 | ProxySQL failover | 자동 | | ✅/❌ |
| 4 | 데이터 손실 | 0건 | | ✅/❌ |
| 5 | 애플리케이션 자동 재연결 | 성공 | | ✅/❌ |
| 6 | 복제 상태 | Normal | | ✅/❌ |
| 7 | 알람 발송 | 정상 | | ✅/❌ |
| 8 | 복제 지연 | < 1초 | | ✅/❌ |

---

## 7. 주의사항

⚠️ **테스트 전 필수 확인:**
1. **Backup 확보** — 데이터 손상 시 복구용
2. **유휴 시간대 실행** — 실제 트래픽 영향 최소화
3. **On-Call 준비** — 예상치 못한 장애 대응
4. **모니터링 활성화** — 모든 이벤트 기록
5. **Rollback 계획** — Failover 후 원상복구 절차 준비

---

## 8. 연관 개념

- [[2026-06-13-01_(MySQL-Binary-Log-Position-복제)]]
- [[2026-06-13-02_(MySQL-GTID-기반-복제)]]
- [[2026-06-13-06_(MySQL-Replication-에러-처리-및-복구)]]
