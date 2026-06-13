# MySQL Percona 설치 (상세 가이드)

- **카테고리**: #DBMS #MySQL #Installation
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-install percona by rpm on redhat - BigDataTeam.md]]

## 1. 핵심 요약

RedHat/CentOS 환경에서 Percona MySQL을 RPM으로 설치합니다.
주의점은 **compat-openssl10** 의존성 (libcrypto.so.10/libssl.so.10), 기존 MariaDB 제거, 커널 파라미터 설정입니다.
설치 순서: 공유 라이브러리 → 클라이언트 → 서버 순으로 진행합니다.

---

## 2. 사전 준비

### 2.1 사용자/그룹 생성

```bash
groupadd mysql
useradd -g mysql -d /home/bos -m -s /bin/bash bos
passwd bos
```

### 2.2 디렉터리 생성

```bash
su - bos
mkdir -p mysql/base mysql/binlog
mkdir -p /home/data/trc /home/data/mytmp
mkdir -p /home/data/log/mysql-bin
mkdir -p /home/data/log/innodb-log
mkdir -p /home/data/log/innodb-undo
mkdir -p /home/data/log/relay-bin
chown -R bos:mysql /home/data
chmod 750 /home/data
```

---

## 3. 의존성 해결

### 3.1 libcrypto.so.10 부족 오류

**증상:**
```
error: Failed dependencies:
  libcrypto.so.10()(64bit) is needed
  libssl.so.10()(64bit) is needed
```

**확인:**
```bash
ldconfig -p | grep libcrypto.so
# 현재: libcrypto.so.1.1만 있음
```

**해결:**
```bash
yum whatprovides "*/libcrypto.so.10"
# 결과: compat-openssl10-1:1.0.2o-3.el8.i686

yum install -y compat-openssl10.x86_64
yum install -y openssl-devel
```

### 3.2 기존 MariaDB 제거

```bash
rpm -qa | grep mariadb
# 결과: 기존 MariaDB 패키지 확인

rpm -e mariadb-connector-c  # 제거 필요 시
```

---

## 4. RPM 설치 순서

### Step 1: 공유 라이브러리

```bash
rpm -ivh percona-server-shared-compat-8.0.36-28.1.el8.x86_64.rpm
rpm -ivh percona-server-shared-8.0.36-28.1.el8.x86_64.rpm
```

### Step 2: 개발 및 클라이언트

```bash
rpm -ivh percona-server-devel-8.0.36-28.1.el8.x86_64.rpm
rpm -ivh percona-server-client-8.0.36-28.1.el8.x86_64.rpm
```

### Step 3: 데이터 파일

```bash
rpm -ivh percona-icu-data-files-8.0.36-28.1.el8.x86_64.rpm
```

### Step 4: 서버

```bash
rpm -ivh percona-server-server-8.0.36-28.1.el8.x86_64.rpm
```

**설치 중 프롬프트:**
```
Creating mysql system user...
Creating mysql system user done.
```

---

## 5. 커널 파라미터 설정

### 5.1 파일 디스크립터

```bash
vi /etc/security/limits.conf

# 추가:
mysql soft nofile 65536
mysql hard nofile 65536
```

### 5.2 재부팅 또는 ulimit 설정

```bash
su - mysql
ulimit -n 65536
```

---

## 6. 초기 설정

### 6.1 my.cnf 편집

```ini
[mysqld]
datadir=/home/bos/mysql/base
log-bin=/home/bos/mysql/binlog/mysql-bin
socket=/tmp/mysql.sock
port=3306

# InnoDB 경로
innodb_data_home_dir=/home/bos/mysql/base
innodb_log_group_home_dir=/home/bos/mysql/log/innodb-log
```

### 6.2 데이터베이스 초기화

```bash
mysqld --defaults-file=/path/to/my.cnf --initialize-insecure \
  --datadir=/home/bos/mysql/base
```

### 6.3 MySQL 시작

```bash
systemctl start mysqld.service
mysql -uroot -p  # 비밀번호 없음
```

---

## 7. 초기 보안 설정

```sql
-- root 비밀번호 설정
ALTER USER 'root'@'localhost' IDENTIFIED BY 'newpassword';

-- 삭제할 사용자/권한
DROP USER ''@'localhost';
DROP USER ''@'hostname';
DROP DATABASE test;

FLUSH PRIVILEGES;
```

---

## 8. 체크리스트

| 항목 | 확인 |
|------|------|
| **compat-openssl10** | ✅ 설치됨 |
| **MariaDB 제거** | ✅ 충돌 없음 |
| **RPM 설치 순서** | ✅ 라이브러리 → 클라이언트 → 서버 |
| **파일 디스크립터** | ✅ 65536 이상 |
| **데이터 디렉터리** | ✅ 권한 설정 (mysql:mysql) |
| **MySQL 시작** | ✅ 정상 작동 |

---

## 9. 연관 개념

- [[2026-06-13-12_(MySQL-Percona-V8-Full-Backup-스크립트)]]
