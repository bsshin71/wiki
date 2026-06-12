# MySQL Percona Server 설치 가이드
- **카테고리**: #DBMS #MySQL
- **태그**: #MySQL #Percona #install #my.cnf #설정 #경로수정 #패치 #커널파라미터
- **작성일**: 2026-06-12
- **참조 원본**: [[2026-06-12-install percona by rpm on redhat - BigDataTeam]], [[2026-06-12-install percona by rpm on redhat(설치요약본) - BigDataTeam]], [[2026-06-12-percona mysql devel 환경만들기 - BigDataTeam]], [[2026-06-12-Percona MySQL 경로 수정 - BigDataTeam]], [[2026-06-12-percona server 8.0.40 패치과정 - BigDataTeam]], [[2026-06-12-percona mysql 8.4  vs percona mysql 8.0 - BigDataTeam]], [[2026-06-12-prd db용 my.cnf - BigDataTeam]], [[2026-06-12-성능 최적화를 위한 주요 커널파라미터 - BigDataTeam]]

## 1. 핵심 요약
- RHEL/Rocky Linux 8에서 Percona Server RPM 설치 시 **libcrypto.so.10 호환 라이브러리** 설치가 핵심.
- datadir 변경 시 `/etc/my.cnf`에 명시하고, `mysqld_pre_systemd` 스크립트도 수정 필요 (system table 중복 방지).
- **Percona 8.4 주의**: `SLAVE`/`MASTER` 명령어가 `REPLICA`/`SOURCE`로 변경. MHA와 연동 불가. ProxySQL은 2.6.0+에서 `caching_sha2_password` 지원.
- `mysql_native_password` 플러그인이 8.4에서 기본 비활성화됨 → 수동 로드 필요.

## 2. 상세 설명

### RPM 설치 절차 (RHEL 8)

```bash
# 1. OS 계정·디렉토리 준비
groupadd mysql
useradd -g mysql -d /home/bos -m -s /bin/bash bos
passwd bos
su - bos
mkdir -p ~/mysql/{base,binlog}

# 2. libcrypto.so.10 호환 라이브러리 설치 (RHEL 8 필수)
yum whatprovides "*/libcrypto.so.10"
# compat-openssl10-1:1.0.2o-3.el8.x86_64

yum makecache --refresh
yum install compat-openssl10-1:1.0.2o-3.el8.x86_64

# libcrypto.so.10 확인
ldconfig -p | grep libcrypto.so
# libcrypto.so.10 (libc6,x86-64) => /lib64/libcrypto.so.10

# 3. RPM 설치 (순서 중요)
rpm -ivh percona-server-shared-compat-8.0.34-26.1.el8.x86_64.rpm
rpm -ivh percona-server-shared-8.0.34-26.1.el8.x86_64.rpm
rpm -ivh percona-server-client-8.0.34-26.1.el8.x86_64.rpm
rpm -ivh percona-icu-data-files-8.0.34-26.1.el8.x86_64.rpm
rpm -ivh percona-server-server-8.0.34-26.1.el8.x86_64.rpm

# 4. SELinux 비활성화 (mysqld 시작 오류 방지)
setenforce 0
sestatus    # Current mode: permissive 확인
# 영구 비활성화: /etc/selinux/config → SELINUX=permissive
# SELinux fcontext 오류 발생 시
chcon -t bin_t /usr/sbin/mysqld

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

### Percona 8.0.40 패치 과정 (버전 업그레이드)

```bash
systemctl stop mysql

# RPM 업그레이드 (--force 옵션)
rpm -Uvh percona-server-shared-compat-8.0.40-31.1.el8.x86_64.rpm --force
rpm -Uvh percona-server-shared-8.0.40-31.1.el8.x86_64.rpm --force
rpm -Uvh percona-server-devel-8.0.40-31.1.el8.x86_64.rpm --force
rpm -Uvh percona-server-client-8.0.40-31.1.el8.x86_64.rpm --force
rpm -Uvh percona-icu-data-files-8.0.40-31.1.el8.x86_64.rpm --force
rpm -Uvh percona-server-server-8.0.40-31.1.el8.x86_64.rpm --force
# 오류: libatomic.so.1 needed, percona-telemetry-agent needed

# libatomic 설치
yum install libatomic

# percona-telemetry-agent 설치 후 비활성화
rpm -ivh percona-telemetry-agent-1.0.1-1.el8.x86_64.rpm
systemctl stop percona-telemetry-agent
systemctl disable percona-telemetry-agent

# 또는 설치 시 텔레메트리 비활성화
PERCONA_TELEMETRY_DISABLE=1 yum install percona-server-server

rpm -Uvh percona-server-server-8.0.40-31.1.el8.x86_64.rpm --force
systemctl start mysql

# 버전 확인
mysql> SELECT version();  -- 8.0.40-31
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
GRANT SUPER ON *.* TO `root`@`%` WITH GRANT OPTION;

-- 일반 계정 생성 (mysql_native_password 명시)
CREATE USER `bos`@`%` IDENTIFIED WITH mysql_native_password BY 'xxxx';
CREATE DATABASE TESTDB;
GRANT ALL PRIVILEGES ON TESTDB.* TO `bos`@`%`;
FLUSH PRIVILEGES;
```

---

### datadir 경로 수정 (기본 /var/lib/mysql → 커스텀 경로)

```bash
# 1. 새 디렉토리 생성 및 권한
mkdir -p /data/mysql
chown -R mysql:mysql /data/mysql
chmod 750 /data/mysql

# 2. /etc/my.cnf 수정
[mysqld]
datadir=/data/mysql
basedir=/opt/percona
socket=/data/mysql/mysql.sock
log-error=/data/mysql/mysqld.log
pid-file=/data/mysql/mysqld.pid

# 3. 기존 디렉토리 백업
mv /var/lib/mysql /var/lib/mysql.bak

# 4. DB 초기화
mysqld --defaults-file=/etc/my.cnf --initialize-insecure --user=mysql

# 5. systemd 서비스 파일 수정
# /etc/sysconfig/mysql에 추가
DEFAULTFILE="--defaults-file=/home/mysql/mysql_home/etc/my.cnf"
MYSQLD_OPTS="--defaults-file=/home/mysql/mysql_home/etc/my.cnf"

# /usr/lib/systemd/system/mysqld.service 수정
ExecStart=/usr/sbin/mysqld $MYSQLD_OPTS

# /usr/bin/mysqld_pre_systemd 수정 (get_option 함수 내)
[ -e /etc/sysconfig/mysql ] && . /etc/sysconfig/mysql    # 이 줄 추가
ret=$(/usr/bin/my_print_defaults ${DEFAULTFILE} ...

systemctl daemon-reload
systemctl start mysql
systemctl enable mysql
```

> ⚠️ `/etc/my.cnf`가 아닌 다른 경로의 my.cnf를 사용할 경우 system table이 `/var/lib/mysql`과 지정된 datadir에 중복 생성될 수 있음. 반드시 `/etc/my.cnf`에 datadir 명시 권장.

---

### 운영용 my.cnf 전체 예시 (Percona 8.0)

```ini
[mysqld]
server-id                 = 51
basedir                   = /home/mysql/mysql_home
datadir                   = /home/data/mysql
default_storage_engine    = innodb
table_open_cache          = 4000
open_files_limit          = 65535
socket                    = /var/lib/mysql/mysql.sock
local_infile              = OFF
pid-file                  = /home/data/trc/mysqld.pid

# Transaction
transaction-isolation     = READ-COMMITTED

# MHA/Replication
event_scheduler           = OFF
sql_mode=ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
relay_log_purge           = 0
log_bin_trust_function_creators = 1
expire_logs_days          = 1
sysdate-is-now
default-time-zone         = '+00:00'

## DB Connection
character-set-server      = utf8mb4
character-set-filesystem  = utf8mb4
collation-server          = utf8mb4_0900_ai_ci
skip-character-set-client-handshake
max_connections           = 600
max_user_connections      = 600
max_connect_errors        = 999999
skip-name-resolve
connect_timeout           = 20
max_allowed_packet        = 32M
interactive_timeout       = 86400
wait_timeout              = 31536000
autocommit                = 1

## MySQL BinLog
log-bin                   = /home/data/log/mysql-bin/mysql-bin
sync_binlog               = 1
enforce_gtid_consistency  = ON
gtid_mode                 = ON
binlog_format             = row
binlog_row_image          = FULL
max_binlog_size           = 104857600
# binlog_expire_logs_seconds = 1209600  # 14일

## Relay Log
relay-log                 = /home/data/log/relay-bin/relay-bin
relay_log_info_repository = TABLE
relay_log_recovery        = ON
relay_log_purge           = ON

## InnoDB
innodb_sort_buffer_size   = 64M
innodb_data_home_dir      = /home/data/mysql
innodb_data_file_path     = ibdata1:100M:autoextend
innodb_temp_data_file_path= ibtmp1:12M:autoextend
innodb_log_group_home_dir = /home/data/log/innodb-log
innodb_log_files_in_group = 3
innodb_log_file_size      = 2048M
innodb_file_per_table     = ON
innodb_undo_directory     = /home/data/log/innodb-undo
innodb_rollback_segments  = 64
innodb_undo_tablespaces   = 2
innodb_max_undo_log_size  = 536870912
innodb_undo_log_truncate  = ON
innodb_flush_log_at_trx_commit = 1
innodb_buffer_pool_size   = 8G
innodb_buffer_pool_instances = 8
innodb_doublewrite        = OFF
innodb_status_output_locks = ON
innodb_print_all_deadlocks = ON
innodb_adaptive_hash_index = ON
innodb_buffer_pool_in_core_file = OFF

## Error Log
log-error                 = /home/data/trc/mysql-err.log
log_error_verbosity       = 3
log_error_suppression_list= 'MY-013360'

## Slow Query Log (Percona 확장)
slow_query_log            = ON
long_query_time           = 3
log_slow_rate_limit       = 1
log_slow_rate_type        = query
log_slow_verbosity        = full
log_slow_admin_statements = ON
log_slow_slave_statements = ON
slow_query_log_always_write_time = 1
slow_query_log_use_global_control = all
slow_query_log_file       = /home/data/trc/mysql-query.log

## Replication
log_slave_updates         = ON
session_track_gtids       = OWN_GTID
slave_parallel_type       = LOGICAL_CLOCK
slave_parallel_workers    = 1
slave_preserve_commit_order = 1
binlog_rows_query_log_events = ON

## Monitoring
userstat                  = 1
read_only                 = 0

## Validate Password (설치 후 활성화)
# validate_password_policy = LOW
# validate_password_length = 4
# default_authentication_plugin = mysql_native_password
```

---

### Linux 커널 파라미터 최적화 (`/etc/sysctl.conf`)

```bash
# 파일 디스크립터
fs.file-max = 100000              # 시스템 전체 최대 파일 수
fs.aio-max-nr = 1048576           # 비동기 I/O 최대 요청 수 (InnoDB)

# 메모리
vm.swappiness = 10                # Swap 최소화 (MySQL 메모리 확보)
vm.dirty_ratio = 15               # dirty page 비율
vm.dirty_background_ratio = 5    # 백그라운드 write 비율
vm.overcommit_memory = 1          # 메모리 오버커밋 허용

# 네트워크
net.core.somaxconn = 4096         # 동시 접속 요청 큐
net.core.netdev_max_backlog = 5000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_fin_timeout = 15     # TIME_WAIT 빠른 정리
net.ipv4.tcp_tw_reuse = 1         # TIME_WAIT 소켓 재사용

# 적용
sudo sysctl -p
```

**파일 디스크립터 설정** (`/etc/security/limits.conf`):
```
mysql soft nofile 65535
mysql hard nofile 65535
```

---

### Percona 8.0 vs 8.4 주요 차이

| 항목 | 8.0 | 8.4 |
|------|-----|-----|
| 릴리스 모델 | 패치마다 새 기능 포함 | LTS (Long Term Support) |
| mysql_native_password | 기본 활성화 | **기본 비활성화 (수동 로드 필요)** |
| Replication 명령어 | SLAVE/MASTER | **REPLICA/SOURCE** |
| mysqlpump/mysql_upgrade | 있음 | **제거됨** |
| ProxySQL 연동 | 용이 | caching_sha2_password 복잡성 |
| MHA 연동 | 가능 | **호환성 문제 (명령어 변경)** |
| UUID_VX 컴포넌트 | 없음 | 추가됨 |

> 💡 현재(2026-06) 권장: ProxySQL·MHA 연동을 위해 **Percona 8.0 최신 버전** 사용. 8.4는 MHA 호환성 확인 후 도입.

## 3. 연관 개념 (지식 연결)
- [[2026-06-12_(MySQL-XtraBackup-백업복구-가이드)]] — XtraBackup 설치 및 백업 계정 설정
- [[2026-06-12_(MySQL-Replication-가이드)]] — my.cnf Replication 설정 활용
- [[2026-06-12_(MySQL-InnoDB-구조-설정)]] — InnoDB 파라미터 상세 설명
