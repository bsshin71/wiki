# MySQL Percona 설치 (요약 체크리스트)

- **카테고리**: #DBMS #MySQL #Installation
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-install percona by rpm on redhat(설치요약본) - BigDataTeam.md]]

## 1. 핵심 요약

Percona MySQL 설치를 빠르게 진행하기 위한 체크리스트 형식의 가이드입니다.
사전 준비 → 패키지 설치 → 초기화 → 시작의 4단계로 구성되며, 각 단계마다 검증 포인트를 명시합니다.

---

## 2. 사전 준비 (5분)

```bash
# 1단계: 사용자/그룹 생성
groupadd mysql
useradd -g mysql -d /home/mysql -m -s /bin/bash mysql
passwd mysql

# 2단계: 데이터 디렉터리 생성
mkdir -p /home/data/trc
mkdir -p /home/data/mytmp
mkdir -p /home/data/log/mysql-bin
mkdir -p /home/data/log/innodb-log
mkdir -p /home/data/log/innodb-undo
mkdir -p /home/data/log/relay-bin
chown -R mysql:mysql /home/data
chmod 750 /home/data

# 3단계: 커널 파라미터 설정
vi /etc/security/limits.conf
# 추가:
# mysql soft nofile 65536
# mysql hard nofile 65536
```

---

## 3. 의존성 설치 (5분)

```bash
# OpenSSL 호환성 라이브러리
yum -y install compat-openssl10.x86_64
yum -y install openssl-devel

# MariaDB 제거 (충돌 가능)
rpm -qa | grep mariadb
```

---

## 4. RPM 설치 (10분)

### 4.1 클라이언트 라이브러리

```bash
rpm -ivh percona-server-shared-compat-8.0.36-28.1.el8.x86_64.rpm
rpm -ivh percona-server-shared-8.0.36-28.1.el8.x86_64.rpm
rpm -ivh percona-server-devel-8.0.36-28.1.el8.x86_64.rpm
rpm -ivh percona-server-client-8.0.36-28.1.el8.x86_64.rpm
```

### 4.2 서버

```bash
rpm -ivh percona-icu-data-files-8.0.36-28.1.el8.x86_64.rpm
rpm -ivh percona-server-server-8.0.36-28.1.el8.x86_64.rpm
```

**검증:**
```bash
mysql --version
# Percona Server (GPL), version 8.0.36-28.1 (Release)
```

---

## 5. 초기화 및 시작 (5분)

```bash
# 1단계: MySQL 초기화
mysqld --initialize-insecure --datadir=/home/data --user=mysql

# 2단계: MySQL 시작
systemctl start mysqld.service
systemctl status mysqld.service

# 3단계: 접속 확인
mysql -uroot  # 비밀번호 없음

# 4단계: 비밀번호 설정
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'newpass';
mysql> FLUSH PRIVILEGES;
```

---

## 6. 빠른 검증

```bash
# MySQL 실행 확인
mysql -uroot -p -e "SELECT VERSION();"

# 데이터 디렉터리 확인
ls -la /home/data/

# 권한 확인
ls -la /var/lib/mysql/ | head
```

---

## 7. 설치 흐름도

```
사전준비 (사용자, 디렉터리, 커널)
    ↓
의존성 (compat-openssl10)
    ↓
RPM 설치 (라이브러리 → 클라이언트 → 서버 순)
    ↓
초기화 (--initialize-insecure)
    ↓
시작 (systemctl start)
    ↓
검증 (mysql -V, SELECT VERSION())
```

---

## 8. 문제해결

| 문제 | 증상 | 해결 |
|------|------|------|
| OpenSSL 부족 | `libcrypto.so.10` 오류 | `yum install compat-openssl10` |
| MariaDB 충돌 | `mariadb-connector-c` 오류 | `rpm -e mariadb-connector-c` |
| 권한 오류 | `Access denied` | `chown -R mysql:mysql /home/data` |
| 초기화 실패 | 데이터 디렉터리 오류 | 디렉터리 삭제 후 재실행 |

---

## 9. 설치 시간 추정

| 단계 | 시간 |
|------|------|
| 사전 준비 | 5분 |
| 의존성 설치 | 5분 |
| RPM 설치 | 10분 |
| 초기화 및 시작 | 5분 |
| **총 소요시간** | **~25분** |

---

## 10. 연관 개념

- [[2026-06-13-14_(MySQL-Percona-설치-상세-가이드)]]
- [[2026-06-13-12_(MySQL-Percona-V8-Full-Backup-스크립트)]]
