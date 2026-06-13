# MySQL MHA 구축 (post-install 설정)

- **카테고리**: #DBMS #MySQL #HA
- **태그**: #HA #MHA #VIP #failover #alarm
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-MHA 구축(post install 위주) - BigDataTeam]]

## 1. 핵심 요약

MHA 컴파일·설치 이후의 **운영 설정** 과정으로, config 파일 작성·VIP 전환 스크립트·MySQL 8.0 syntax 코드 수정·purge relay log·failover 알람을 다룹니다.
편의 스크립트 `manager.sh`로 start/stop/switch/failback/status를 단순화하고, `send_report`→`send_alarm.sh`로 failover 알람을 발송합니다.

---

## 2. 디렉토리 구조

```
/mha/node        # MHA node 실행파일
/mha/manager     # MHA manager 실행파일
/mha/managerwork # conf, script, log, work_dir
/mha/mhawork     # node 작업경로(log/script/tmp)
```

## 3. config 파일

- **공용 마스터**: `/etc/masterha_default.cnf` (user/password/ssh_port/repl_user/master_binlog_dir) — `chown mha:mysql`
- **서비스별**: `/mha/managerwork/conf/cfddb.conf` — `secondary_check_script`, `master_ip_failover_script`, `report_script`, `master_ip_online_change_script`, server1/2 `candidate_master=1`

## 4. ⭐ MySQL 8.0 호환 코드 수정

| 파일 | 수정 |
|------|------|
| `NodeUtil.pm` | 버전 `-` 파싱 (`($str)=$str=~m/^[^-]*/g`) |
| `master_ip_online_change` | 작동 안 하는 가상코드 삭제 + VIP 변경 스크립트 호출 추가 |
| `master_ip_failover` | `ssh_port` 옵션 추가 + VIP 변경 호출 |
| `DBHelper.pm` | `CHANGE MASTER`→`CHANGE REPLICATION SOURCE`, `START/STOP SLAVE`→`REPLICA` 변경 |

> ⚠️ **`show slave status`는 절대 `show replica status`로 바꾸지 말 것** — MHA가 slave를 인식 못해 동작 실패.

## 5. VIP 전환 스크립트 (mha_change_vip.sh)

```bash
# 신규 master로 VIP 재배치: 기존 VIP down(ssh) → 신규 master에 ifconfig up + arping
ssh $V_NEW_MASTER /bin/sudo /sbin/ifconfig ${NET_INTERFACE_ID} $V_VIP_IP netmask ... up
ssh $V_NEW_MASTER /sbin/arping -c 5 -D -I ${NET_INTERFACE} -s $V_VIP_IP $V_VIP_IP
```

## 6. 편의 스크립트 (manager.sh)

```bash
./manager.sh cfd start     # nohup masterha_manager 기동
./manager.sh cfd status    # PING_OK, master 확인
./manager.sh cfd checkrep  # 이중화(master/slave) 상태
./manager.sh cfd checkssh  # ssh 연결 점검
./manager.sh cfd failback  # 마스터 복구 후 원복 (old master가 slave로 설정돼 있어야 함)
```

## 7. purge relay log

```bash
# my.cnf: relay_log_purge = 0  (MHA가 relay log 직접 관리)
# DB 노드에서 crontab으로 주기 실행
0 1 * * * /mha/mhawork/script/auto_clean_relay_log.sh   # purge_relay_logs 호출
```

## 8. Failover 알람

```
failover → send_report → send_alarm.sh → alarm server(curl POST)
```
- `send_alarm.sh` 작성(telegram/slack/email/mobile), `send_report` 수정(send_alarm 호출), config에 `report_script` 등록
- IM/SMS는 단문, 상세는 email

## 9. 연관 개념

- [[2026-06-14-65_(MySQL-MHA-설치-RedHat8)]]
- [[2026-06-14-67_(MySQL-ProxySQL-MHA-연동-VIP-Failover)]]
- [[2026-06-13-34_(MySQL-Replication-재설정)]]
