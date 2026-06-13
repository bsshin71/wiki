# MySQL ProxySQL · MHA 연동 (VIP Failover)

- **카테고리**: #DBMS #MySQL #HA
- **태그**: #HA #ProxySQL #MHA #VIP #failover
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-ProxySQL과 MHA연동 - BigDataTeam]]

## 1. 핵심 요약

MHA online failover 시 `master_ip_failover` 스크립트가 **VIP를 새 Master로 이동**시키고, 애플리케이션은 VIP로 접속하므로 자동으로 새 Master에 연결됩니다.
**DB failover + VIP failover + app의 reconnect/retry 로직**이 모두 정상 동작하면 데이터 유실 없이 failover가 완료됩니다.

---

## 2. 동작 방식 (VIP 기반)

1. MHA Manager가 Master 장애 감지 → `master_ip_failover` 호출
2. 스크립트가 기존 Master의 VIP(`enp0s8:0`) down → 새 Master에 VIP up
   ```bash
   ssh mha@new_master sudo /sbin/ifconfig enp0s8:0 192.168.56.10 netmask 255.255.255.0 up
   ```
3. app은 VIP로 접속하므로 끊김 발생 시 **reconnect 후 재시도**하면 새 Master로 연결
4. app은 serial key insert + connection 오류 시 reconnect 로직으로 **데이터 유실 0** 보장

## 3. Failover 단계 (MHA 로그 흐름)

| Phase | 내용 |
|-------|------|
| 감지 | Master 미응답 4회 → `Master is down! exitcode 20` |
| Phase 1 | Configuration Check (GTID failover mode=1) |
| Phase 2 | Dead Master Shutdown — VIP 비활성(`--command=stopssh`) |
| Phase 3.1 | Latest Slaves 확인 (binlog position/GTID 최신 slave 선정) |
| Phase 3.3 | New Master 결정 — `candidate_master` slave 중 최신 relay log 보유본 승격 |
| Phase 3 | New Master Recovery — `read_only=0`, VIP 활성(`--command=start`) |
| Phase 4 | 나머지 Slave를 새 Master로 `CHANGE MASTER TO ... AUTO_POSITION=1` 병렬 재연결 |

> GTID auto-position 기반이라 SSH/Node 패키지 체크를 건너뛰고 빠르게 진행.

## 4. 검증 포인트

- `manager.sh status` → `PING_OK, master:<신규>`
- 새 Master `read_only=0` 설정 확인
- app retry 로직으로 insert 누락 0 확인

## 5. 연관 개념

- [[2026-06-14-65_(MySQL-MHA-설치-RedHat8)]]
- [[2026-06-14-66_(MySQL-MHA-구축-post-install-설정)]]
- [[2026-06-13-08_(MySQL-HA-Failover-테스트-전략)]]
- [[2026-06-13-02_(MySQL-GTID-기반-복제)]]
