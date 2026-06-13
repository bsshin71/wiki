# MySQL MHA 설치 (RedHat 8)

- **카테고리**: #DBMS #MySQL #HA
- **태그**: #HA #MHA #failover #설치 #perl
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-MHA 설치 on Redhat8 - BigDataTeam]]

## 1. 핵심 요약

MHA(Master High Availability)는 **MHA Node**(모든 DB 서버)와 **MHA Manager**(별도 감시 서버)로 구성되며, Perl 기반이라 RHEL8에서 부족한 Perl 모듈을 Custom yum repo로 보충해 설치합니다.
MySQL 8.0에서는 버전 문자열의 `-` 기호 때문에 `NodeUtil.pm` 파싱 오류가 발생하므로 **코드 수정이 필수**입니다.

---

## 2. 구성

| 역할 | 호스트 | MHA 타입 |
|------|--------|----------|
| Master DB | redhat8-1 | MHA Node |
| Slave DB1/2 | redhat8-2/3 | MHA Node |
| MHA Manager | redhat8-4 | MHA Manager |

- 환경: RHEL 8.4, Percona 8.0.26, MHA 0.57/0.58

## 3. 설치 절차

### 3.1 Perl 모듈 (모든 서버 공통)
```bash
# RHEL8은 일부 Perl 모듈 미지원 → Custom yum repo(vault.centos.org / rpmfind.net alma) 추가
yum install -y perl perl-Module-Install perl-DBD-MySQL perl-Config-Tiny \
  perl-Params-Validate perl-Parallel-ForkManager perl-Log-Dispatch perl-Time-HiRes perl-CPAN
```

### 3.2 MHA Node (모든 서버) / Manager (매니저만)
```bash
tar xvzf mha4mysql-node-0.57.tar.gz && cd mha4mysql-node-0.57
perl Makefile.PL INSTALL_BASE=/mysql/mha   # 설치 경로 지정
make && make install

# Manager는 mha4mysql-manager-0.57로 동일 절차 (매니저 서버만)
```
> make 오류(`MHA::NodeConst missing`) 시: `PERL5LIB`에 `/mysql/mha/lib/perl5` 추가, `perl-ExtUtils-MakeMaker`·`perl-Module-Install` 설치.

## 4. 사전 설정

- **사용자 계정**: `mha` 유저 생성(모든 서버)
- **SSH 무패스워드**: 매니저→모든 노드 키 교환(`ssh-keygen`/`authorized_keys`, `chmod 400`)
- **VIP sudo 권한**: `visudo`에 `mha ALL=(ALL) NOPASSWD:/sbin/ifconfig`
- **DB 접속 계정**: 매니저용 `root@'%'`, 로컬 스크립트용 `root@'127.0.0.1'`
- **방화벽**: `systemctl stop/disable firewalld` (3306 차단 방지)
- **동작 디렉토리**: `/mysql/mha/mhawork/{log,tmp,script}`

## 5. ⭐ MySQL 8.0 호환 코드 수정 (필수)

### 5.1 버전 `-` 파싱 오류 (NodeUtil.pm)
```perl
# /mysql/mha/lib/perl5/MHA/NodeUtil.pm — parse_mysql_version() 등에 추가
($str) = $str =~ m/^[^-]*/g;   # 8.0.34-26 의 '-26' 제거
```
> 미수정 시: `Redundant argument in sprintf at NodeUtil.pm line 184` 발생.

### 5.2 기타
- DBD::mysql 낮은 버전 IPv6 미지원 → 최신 설치 또는 `DBHelper.pm`/`HealthCheck.pm`의 `[]` 제거
- `master_ip_failover`에 `orig_master_ssh_port` 옵션 코드 추가

## 6. 설정 파일

- **글로벌**: `/etc/masterha_default.cnf` (user/password/ssh_user 공통)
- **서비스별**: `bosdb.cnf` (server1~3 hostname, `candidate_master=1`, `ping_interval`, `master_binlog_dir`)

## 7. VIP 설정 & 명령어

```bash
ifconfig enp0s8:0 192.168.138.100 netmask 255.255.255.0 up   # VIP 할당
```
| 명령어 | 역할 |
|--------|------|
| masterha_manager | 매니저 구동(자동 failover) |
| masterha_check_ssh / _repl / _status | SSH·복제·매니저 상태 점검 |
| masterha_master_switch | 수동 failover |
| masterha_stop | 매니저 중단 |

## 8. 연관 개념

- [[2026-06-14-66_(MySQL-MHA-구축-post-install-설정)]]
- [[2026-06-14-67_(MySQL-ProxySQL-MHA-연동-VIP-Failover)]]
- [[2026-06-13-08_(MySQL-HA-Failover-테스트-전략)]]
