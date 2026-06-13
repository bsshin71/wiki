# MySQL Orchestrator 설치 및 특징

- **카테고리**: #DBMS #MySQL #HA
- **태그**: #HA #orchestrator #failover #topology
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Orchestrator 설치 및 특징 - BigDataTeam]]

## 1. 핵심 요약

Orchestrator는 복제 토폴로지를 **시각화(Discovery)·GUI 리팩토링·자동 복구(Recovery)** 하는 MySQL HA 도구입니다.
별도 repository DB에 상태를 저장하며 웹 UI(3000 포트)로 운영합니다. 다만 **죽은 Master의 binlog를 회수하지 않아 data loss 가능성**이 있고, 소스 업데이트가 2021년 7월 이후 중단된 점에 유의합니다.

---

## 2. 주요 기능

- **Discovery**: 토폴로지에서 복제 상태·구성을 지속 수집, 장애 시 시각화
- **Refactoring**: binlog position/GTID/Pseudo-GTID 수집, GUI 드래그로 복제본 재배치(비정상 시도 거부)
- **Recovery**: Master/중간 Master 장애 감지, 상태 기반(구성 아님) 자동/수동 복구
- 기타: HA, Pseudo-GTID, Datacenter awareness, HTTP 인증

## 3. 설치 절차

### 3.1 감시 대상 MySQL 유저
```sql
CREATE USER 'orchestrator'@'%' IDENTIFIED BY 'orchestrator';
GRANT SUPER, PROCESS, REPLICATION SLAVE, RELOAD ON *.* TO 'orchestrator'@'%';
GRANT SELECT ON mysql.slave_master_info TO 'orchestrator'@'%';
```

### 3.2 Repository DB 계정
```sql
CREATE DATABASE IF NOT EXISTS orchestrator;
CREATE USER 'orchestrator'@'127.0.0.1' IDENTIFIED BY 'orchestrator';
GRANT ALL PRIVILEGES ON `orchestrator`.* TO 'orchestrator'@'127.0.0.1';
```

### 3.3 설치 & 설정
```bash
rpm -ivh orchestrator-3.2.6-1.x86_64.rpm
vi /usr/local/orchestrator/orchestrator.conf.json
#   MySQLOrchestrator* (repository 접속), MySQLTopology* (감시 대상 접속) 수정
mkdir -p /usr/local/orchestrator/log
```

### 3.4 hostname 해석
```json
// hostname 사용
"HostnameResolveMethod": "default",
"MySQLHostnameResolveMethod": "@@hostname",
// IP 사용
"HostnameResolveMethod": "none",
"MySQLHostnameResolveMethod": "",
```

### 3.5 시작 & 접속
```bash
systemctl start orchestrator      # 또는 ./orchestrator http
systemctl stop/disable firewalld  # 3000 포트 허용
# 웹: http://ip:3000
```

## 4. ⚠️ 한계 (Data Loss 가능성)

- MHA와 달리 **죽은 master에서 binlog를 회수해 동기화하는 단계가 없음**.
- 새 Master 후보 선정 시 `Read Master log Pos`가 아닌 **`Executed Gtid Set`** 기준.
- 극한 시나리오에서 데이터 유실 방지가 필요하면 **MHA가 더 안전**.
- 소스 업데이트 중단(마지막 릴리즈 2021-07).

## 5. 연관 개념

- [[2026-06-14-69_(MySQL-Orchestrator-Failover)]]
- [[2026-06-14-65_(MySQL-MHA-설치-RedHat8)]]
- [[2026-06-13-08_(MySQL-HA-Failover-테스트-전략)]]
