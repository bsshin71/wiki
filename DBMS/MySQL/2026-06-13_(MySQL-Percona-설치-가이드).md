# MySQL Percona Server 설치 가이드

- **카테고리**: #DBMS #MySQL #Percona #Install
- **작성일**: 2026-06-13
- **참조 원본**: 8개 Percona 관련 소스 파일

## 1. 핵심 요약

Percona Server는 MySQL의 drop-in replacement로, **성능 최적화, 버그 픽스, 운영 편의 기능**을 제공합니다.
RHEL/Rocky Linux에 RPM으로 설치하며, 설치 후 기본 my.cnf 설정으로 운영 환경을 구성합니다.
기본 MySQL과 호환되지만 일부 추가 기능(Performance Schema 확장, XtraBackup 통합)을 활용할 수 있습니다.

---

## 2. 사전 준비

### 2.1 시스템 요구사항

- **OS**: RHEL 8+, Rocky Linux 8+, CentOS 8+
- **디스크**: 최소 20GB (데이터 영역 별도)
- **메모리**: 최소 2GB (4GB+ 권장)
- **라이브러리**: libcrypto.so.10, libssl.so.10 (OpenSSL 1.0 호환성)

### 2.2 사용자 및 디렉토리 생성

```bash
# MySQL 그룹 및 사용자 생성
groupadd mysql
useradd -g mysql -d /home/bos -m -s /bin/bash bos
passwd bos

# 데이터 디렉토리 생성
mkdir -p /home/bos/mysql/{base,binlog,backup}
chown -R bos:mysql /home/bos/mysql
chmod 755 /home/bos/mysql
```

---

## 3. libcrypto 호환성 해결

### 3.1 문제 증상

```
오류: Failed dependencies:
  libcrypto.so.10()(64bit) is needed
  libssl.so.10()(64bit) is needed
```

### 3.2 해결 방법

```bash
# OpenSSL 1.0.2 호환성 패키지 설치
rpm -ivh openssl10-libs-1.0.2k-21.el8.x86_64.rpm

# 또는 yum 리포지토리에서
yum install openssl10-libs openssl10

# 설치 확인
ldconfig -p | grep libcrypto.so.10
```

---

## 4. RPM 설치 절차

### 4.1 Percona 리포지토리 등록

```bash
# Percona 공식 RPM 다운로드
rpm -ivh https://repo.percona.com/yum/percona-release-latest.noarch.rpm

# 또는 로컬에서
rpm -ivh percona-release-1.0-27.noarch.rpm
```

### 4.2 Percona Server 설치

```bash
# Percona 패키지 정보 확인
yum search percona-server

# 특정 버전 설치 (예: 8.0.34)
yum install percona-server-server-8.0.34-26

# 또는 최신 버전
yum install percona-server-server

# 의존성 자동 설치됨
# - percona-server-shared
# - percona-server-client
# - percona-server-shared-compat (1.0 호환성)
```

### 4.3 설치 확인

```bash
# 버전 확인
mysql --version
mysqld --version

# 설치된 RPM 조회
rpm -qa | grep percona-server
```

---

## 5. 기본 구성 (my.cnf)

### 5.1 운영 환경용 my.cnf 예시

```ini
[mysqld]
# 기본 설정
port=3306
socket=/tmp/mysql.sock
datadir=/home/bos/mysql/base
tmpdir=/home/bos/mysql/tmp
skip-external-locking

# 서버 ID (복제용)
server-id=1

# Binary Log 설정
log-bin=/home/bos/mysql/binlog/mysql-bin
binlog_format=ROW
expire_logs_days=7

# 로그 경로
log-error=/var/log/mysql/error.log
log_warnings=2

# InnoDB 설정
default-storage-engine=InnoDB
innodb_buffer_pool_size=4G        # 메모리의 50-80%
innodb_log_file_size=512M
innodb_flush_log_at_trx_commit=1  # 내구성 우선
sync_binlog=1
innodb_file_per_table=ON
innodb_flush_method=O_DIRECT

# 성능 관련
max_connections=500
max_allowed_packet=64M
table_open_cache=2000
tmp_table_size=32M
max_heap_table_size=32M
query_cache_type=0           # 캐시 비활성화 (5.7+)
slow_query_log=1
slow_query_log_file=/var/log/mysql/slow.log
long_query_time=2

# Performance Schema (Percona 확장)
performance_schema=ON
performance_schema_max_table_instances=12500

# GTID (복제용)
gtid_mode=ON
enforce_gtid_consistency=ON

# 보안
skip-name-resolve

[mysqld_safe]
log-error=/var/log/mysql/error.log
pid-file=/var/run/mysql/mysql.pid

[mysql]
no-auto-rehash

[mysqldump]
quick
quote-names
max_allowed_packet=64M
```

### 5.2 my.cnf 설치

```bash
# 기본 my.cnf 백업
cp /etc/my.cnf /etc/my.cnf.bak

# 새 my.cnf 작성
cat > /etc/my.cnf << 'EOF'
[위의 내용]
EOF

# 권한 설정
chmod 644 /etc/my.cnf
```

---

## 6. 서비스 시작 및 초기화

### 6.1 디렉토리 권한 설정

```bash
mkdir -p /home/bos/mysql/tmp
chown -R bos:mysql /home/bos/mysql
chmod 700 /home/bos/mysql/base
chmod 755 /home/bos/mysql/{binlog,tmp,backup}
```

### 6.2 MySQL 서비스 시작

```bash
# 서비스 시작
systemctl start mysql
systemctl enable mysql

# 상태 확인
systemctl status mysql
```

### 6.3 초기 접속 확인

```bash
# 기본 비밀번호 확인 (설치 시 생성)
cat /var/log/mysql/error.log | grep "temporary password"

# 접속
mysql -uroot -p

# 비밀번호 변경
ALTER USER 'root'@'localhost' IDENTIFIED BY 'new_password';
FLUSH PRIVILEGES;
```

---

## 7. Percona 특화 기능

### 7.1 XtraBackup 통합

```bash
# Percona Server 설치 시 자동 포함
percona-xtrabackup-80

# 사용 가능
innobackupex --defaults-file=/etc/my.cnf /backup/path
```

### 7.2 Performance Schema 확장

```sql
-- 추가 테이블 확인
SHOW TABLES FROM performance_schema LIKE '%innodb%';
```

### 7.3 Percona 전용 시스템 변수

```sql
SHOW VARIABLES LIKE '%percona%';
SHOW VARIABLES LIKE '%xtradb%';
```

---

## 8. Percona MySQL vs. Oracle MySQL 비교

| 항목 | Percona | Oracle MySQL |
|------|---------|-------------|
| **성능** | 최적화됨 (XtraDB) | 기본 |
| **Bug Fix** | 빠름 (패치 우선) | 정기 릴리스 |
| **XtraBackup** | 기본 포함 | 별도 설치 |
| **Performance Schema** | 확장 버전 | 기본 |
| **커뮤니티 지원** | 활발 | 상업 지원 |
| **라이선스** | 오픈소스 (GPL) | 상업/커뮤니티 |

---

## 9. Percona 8.0 vs 8.4 비교

| 항목 | 8.0 | 8.4 |
|------|-----|-----|
| **MySQL Base** | 5.7 기반 | 8.0 후속 |
| **성능** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **HA 기능** | 표준 | 강화됨 |
| **보안 기능** | 기본 | 확대 |
| **호환성** | 매우 높음 | 높음 |
| **권장** | 안정성 우선 | 최신 기능 우선 |

---

## 10. 개발 환경 구성 (Percona Devel)

### 10.1 Devel 환경 디렉토리

```bash
mkdir -p /home/bos/percona/mysql/{base,binlog,tmp}
chmod 700 /home/bos/percona/mysql/base
chown -R bos:bos /home/bos/percona
```

### 10.2 다중 인스턴스 설정

```ini
# /etc/mysql/mysql_devel.cnf
[mysqld]
port=3307          # 포트 변경
datadir=/home/bos/percona/mysql/base
socket=/tmp/mysql_devel.sock
pid-file=/var/run/mysql_devel.pid
log-error=/var/log/mysql/devel_error.log
```

### 10.3 시작/중지

```bash
# Devel 인스턴스 시작
mysqld_safe --defaults-file=/etc/mysql/mysql_devel.cnf &

# 접속
mysql -S /tmp/mysql_devel.sock -u root

# 중지
mysqladmin -S /tmp/mysql_devel.sock shutdown
```

---

## 11. 패치 및 업그레이드

### 11.1 Percona 8.0.40 패치 예시

```bash
# 현재 버전 확인
mysql -u root -p -e "SELECT VERSION();"

# YUM을 통해 업그레이드
yum update percona-server-server

# 또는 수동 RPM
rpm -Uvh percona-server-server-8.0.40-26.el8.x86_64.rpm
```

### 11.2 업그레이드 후 확인

```bash
# MySQL 재시작
systemctl restart mysql

# 버전 확인
mysql -V

# 호환성 확인
mysql_upgrade -u root -p
```

---

## 12. 연관 개념

- [[2026-06-13_(MySQL-Lock-Deadlock-모니터링)]]
- [[2026-06-13_(MySQL-XtraBackup-백업복구-가이드)]]
- [[2026-06-13_(MySQL-InnoDB-구조-설정)]]
