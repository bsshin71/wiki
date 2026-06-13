# MySQL Orchestrator Failover

- **카테고리**: #DBMS #MySQL #HA
- **태그**: #HA #orchestrator #failover #failback
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Orchestrator Failover - BigDataTeam]]

## 1. 핵심 요약

Orchestrator의 자동 Failover는 `orchestrator.conf.json`의 **Recover 필터**로 대상 클러스터를 지정하고, Master 장애 시 최신 Slave를 새 Master로 승격합니다.
원복(fail-back)은 양방향 복제 구성 후 GUI에서 토폴로지를 드래그하는 방식이라 **절차가 복잡해 운영 환경에서는 수동 원복이 권장**됩니다.

---

## 2. 자동 Failover 설정 (conf.json)

| 파라미터 | 설명 |
|----------|------|
| `RecoverMasterClusterFilters` | 자동 복구 대상 클러스터 지정(`"*"` 또는 정규식/alias) |
| `RecoverIntermediateMasterClusterFilters` | 중간 Master 복구 여부 |
| `PromotionIgnoreHostnameFilters` | Master 승격 제외 Slave hostname |
| `RecoveryPeriodBlockSeconds` | 복구 후 재복구 차단 시간(default 3600 → 테스트 10) |

> 변경 후 Config Reload 필요.

## 3. 장애 유도 & 자동 Failover

```bash
# Master에서 DB stop
systemctl stop mysql
```
- dead master는 토폴로지에서 분리되고, 최신 Slave(db2)가 새 Master로 자동 승격.

## 4. 원복 (Fail-back) 절차

1. 기존 dead master(db1)를 `read_only`로 변경
2. db1 `systemctl start mysql`
3. db2(현 Master) ← db1 이중화 생성 (양방향)
   ```sql
   CHANGE MASTER TO MASTER_HOST='192.168.138.105', MASTER_USER='repl',
     MASTER_PASSWORD='repl', MASTER_AUTO_POSITION=1;
   ```
4. 복제 lag 반영 대기
5. GUI에서 db1을 db2 위로 drag → db1을 Master로 변경
6. db2 이중화 start

## 5. ⚠️ 주의점

- **원복 절차가 복잡** → production 적용 어려움, 수동 원복 권장.
- client Failover와 연동 시 `read_only` 변경 시점 조정이 어려움.
- 동일 slave가 2개로 보이는 현상(/etc/hosts·hostname resolve 관련, 원인 불명) 발생 가능.

## 6. 연관 개념

- [[2026-06-14-68_(MySQL-Orchestrator-설치-및-특징)]]
- [[2026-06-14-67_(MySQL-ProxySQL-MHA-연동-VIP-Failover)]]
- [[2026-06-13-02_(MySQL-GTID-기반-복제)]]
