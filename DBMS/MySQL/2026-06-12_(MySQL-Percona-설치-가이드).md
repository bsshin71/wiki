# MySQL Percona Server 설치 가이드
- **카테고리**: #DBMS #MySQL
- **태그**: #MySQL #Percona #install #my.cnf #설정
- **작성일**: 2026-06-12
- **참조 원본**: [[2026-06-12-install percona by rpm on redhat - BigDataTeam]], [[2026-06-12-install percona by rpm on redhat(설치요약본) - BigDataTeam]], [[2026-06-12-percona mysql devel 환경만들기 - BigDataTeam]], [[2026-06-12-Percona MySQL 경로 수정 - BigDataTeam]], [[2026-06-12-percona server 8.0.40 패치과정 - BigDataTeam]], [[2026-06-12-percona mysql 8.4  vs percona mysql 8.0 - BigDataTeam]], [[2026-06-12-prd db용 my.cnf - BigDataTeam]]

## 1. 핵심 요약
- RHEL/Rocky Linux 환경에서 Percona Server 8.0을 RPM으로 설치. SELinux 비활성화 및 libcrypto.so.10 호환 라이브러리 설치가 핵심.
- my.cnf는 `/etc/my.cnf`에 위치해야 system table 중복 생성 문제를 회피. datadir 경로를 반드시 my.cnf에 명시.
- 설치 후 validate_password, semi_sync 플러그인 설치 권장.

## 2. 상세 설명

### RPM 설치 절차 (RHEL 8)

```bash
# 1. OS 계정·디렉토리 준비
groupadd mysql
useradd -g mysql -d /home/bos -m -s /bin/bash bos
su - bos
mkdir -p ~/mysql/{base,binlog}

# 2. libcrypto.so.10 호환 라이브러리 설치 (RHEL 8 필수)
yum whatprovides "*/libcrypto.so.10"
yum install compat-openssl10-1:1.0.2o-3.el8.x86_64

# 3. RPM 설치 (순서 중요)
rpm -ivh percona-server-shared-compat-8.0.34-26.1.el8.x86_64.rpm
rpm -ivh percona-server-shared-8.0.34-26.1.el8.x86_64.rpm
rpm -ivh percona-server-client-8.0.34-26.1.el8.x86_64.rpm
rpm -ivh percona-icu-data-files-8.0.34-26.1.el8.x86_64.rpm
rpm -ivh percona-server-server-8.0.34-26.1.el8.x86_64.rpm

# 4. SELinux 비활성화 (mysqld 시작 오류 방지)
setenforce 0
# 영구 적용: /etc/selinux/config → SELINUX=permissive

# 5. DB 초기화
mysqld --defaults-file=/etc/my.cnf --initialize-insecure --user=mysql

# 6. systemd 서비스 사용자 변경
vi /usr/lib/systemd/system/mysqld.service
# [Service]
# User=bos
# Group=mysql
# ExecStart=/usr/sbin/mysqld --defaults-file=/etc/my.cnf

# 7. 시작
systemctl daemon-reload
systemctl start mysqld
```

---

### 설치 후 초기 설정

```sql
-- 플러그인 설치
INSTALL PLUGIN validate_password SONAME 'validate_password.so';
INSTALL PLUGIN rpl_semi_sync_master SONAME 'semisync_master.so';
INSTALL PLUGIN rpl_semi_sync_slave SONAME 'semisync_slave.so';

-- root 외부 접속 허용
CREATE USER `root`@`%` IDENTIFIED BY 'xxxx';
GRANT ALL PRIVILEGES ON *.* TO `root`@`%` WITH GRANT OPTION;

-- 일반 계정 생성
CREATE USER `bos`@`%` IDENTIFIED WITH mysql_native_password BY 'xxxx';
GRANT ALL PRIVILEGES ON TESTDB.* TO `bos`@`%`;
FLUSH PRIVILEGES;
```

---

### 운영용 my.cnf (전체 예시)

```ini
[mysqld]
server-id                 = 51
basedir                   = /home/mysql/mysql_home
datadir                   = /home/data/mysql

## Connection
character-set-server      = utf8mb4
collation-server          = utf8mb4_0900_ai_ci
max_connections           = 600
max_allowed_packet        = 32M
transaction-isolation     = READ-COMMITTED
autocommit                = 1

## BinLog
log-bin                   = /home/data/log/mysql-bin/mysql-bin
sync_binlog               = 1
enforce_gtid_consistency  = ON
gtid_mode                 = ON
binlog_format             = row
binlog_row_image          = FULL
max_binlog_size           = 104857600

## Relay Log
relay-log                 = /home/data/log/relay-bin/relay-bin
relay_log_recovery        = ON
relay_log_purge           = ON

## InnoDB
innodb_buffer_pool_size   = 8G
innodb_buffer_pool_instances = 8
innodb_flush_log_at_trx_commit = 1
innodb_file_per_table     = ON
innodb_log_file_size      = 2048M
innodb_log_files_in_group = 3
innodb_print_all_deadlocks = ON

## Undo
innodb_undo_tablespaces   = 2
innodb_max_undo_log_size  = 536870912
innodb_undo_log_truncate  = ON

## Slow Query Log
slow_query_log            = ON
long_query_time           = 3
log_slow_verbosity        = full
slow_query_log_file       = /home/data/trc/mysql-query.log

## Replication
log_slave_updates         = ON
slave_parallel_type       = LOGICAL_CLOCK
slave_parallel_workers    = 1
slave_preserve_commit_order = 1

## 모니터링
userstat                  = 1
innodb_status_output_locks = ON
```

> ⚠️ `/etc/my.cnf` 대신 다른 경로 사용 시 `/etc/sysconfig/mysql`에서 `--defaults-file` 지정하면 system table 중복 생성 문제 발생. `/etc/my.cnf`에 datadir 직접 명시 권장.

---

### 오류 대처

| 오류 | 원인 | 해결 |
|------|------|------|
| `libcrypto.so.10 needed` | RHEL 8 호환 라이브러리 없음 | `compat-openssl10` 설치 |
| SELinux write access 차단 | SELinux enforcing 모드 | `setenforce 0` 또는 fcontext 설정 |
| `caching_sha2_password` 인증 오류 | 기본 인증 플러그인 변경 | `ALTER USER ... IDENTIFIED WITH mysql_native_password` |

## 3. 연관 개념 (지식 연결)
- [[2026-06-12_(MySQL-XtraBackup-백업복구-가이드)]] — XtraBackup 설치 및 백업 계정 설정
- [[2026-06-12_(MySQL-Replication-가이드)]] — my.cnf Replication 설정 활용
- [[2026-06-12_(MySQL-InnoDB-구조-설정)]] — InnoDB 파라미터 상세
