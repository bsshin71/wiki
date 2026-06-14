# PostgreSQL 소스 컴파일 설치

- **카테고리**: #DBMS #PostgreSQL #install
- **태그**: #PostgreSQL #install #소스설치 #컴파일 #systemd
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-Posgresql  소스설치(new) - BigDataTeam]]

## 1. 핵심 요약

PostgreSQL을 소스 컴파일(`./configure → make → make install`)로 설치하는 절차로, **RHEL OS 버전별로 빌드 가능 버전이 다릅니다**(RHEL 8.4는 PG15+ make 무한루프 → 8.8/8.9 업그레이드 또는 yum 설치).
설치 후 `initdb` → systemd 서비스 등록 → `postgresql.conf`(listen_addresses)·`pg_hba.conf`(md5) 변경 + 5432 방화벽 개방이 핵심입니다.

> ※ RPM/yum 설치는 [[2026-06-02_(PostgreSQL-RPM-설치-가이드)]] 참조(본 문서는 **소스 컴파일** 방식).

---

## 2. ⚠️ OS 버전별 컴파일 결과 (RHEL 8.4)

| PG 소스 | 결과 | 비고 |
|---------|------|------|
| 16.1 / 15.5 | **실패** | make 단계에서 configure 무한 반복 (RHEL 8.4). 8.9에서는 15.5 성공 |
| 14.5 / 13.9 / 12.3 | 성공 | |

> RHEL 8.4에서 PG15+ 필요 시 OS를 8.8/8.9로 업그레이드(`yum update`)하거나 yum/dnf 설치로 대체.

## 3. 설치 절차

```bash
# 유저 생성
groupadd -g 26 postgres
useradd -u 26 -g postgres -d /home/postgres postgres

# 빌드 패키지
yum install gcc zlib-devel readline-devel libicu-devel openssl-devel python3-devel autoconf
yum groupinstall 'Development Tools'

# 소스 다운로드·컴파일
wget https://ftp.postgresql.org/pub/source/v14.5/postgresql-14.5.tar.gz
tar xvzf postgresql-14.5.tar.gz && cd postgresql-14.5
./configure --prefix=/home/postgres/pgsql14 --with-python --with-openssl
make && make install

# DB 초기화
./bin/initdb -D /home/postgres/pgsql14/data
```

## 4. systemd 서비스 등록

```ini
# /etc/systemd/system/postgresql.service
[Service]
Type=forking
User=postgres
Environment=PGDATA=/home/postgres/pgsql14/data
ExecStart=/home/postgres/pgsql14/bin/pg_ctl start -D "${PGDATA}" -l "${PGDATA}/logfile"
ExecStop=/home/postgres/pgsql14/bin/pg_ctl stop -D "${PGDATA}"
```
```bash
systemctl enable postgresql && systemctl start postgresql
```

## 5. 접속 설정

```bash
# 방화벽 5432 개방
firewall-cmd --zone=public --permanent --add-port=5432/tcp && firewall-cmd --reload
```
```conf
# postgresql.conf
listen_addresses = '*'              # 외부 접속 허용
# pg_hba.conf
host  all  all  0.0.0.0/0  md5      # 원격 인증
```
```sql
ALTER ROLE postgres PASSWORD 'xxxxx';   -- 비밀번호 설정
```
```bash
systemctl restart postgresql
```

## 6. 연관 개념

- [[2026-06-02_(PostgreSQL-RPM-설치-가이드)]]
- [[2026-06-14-PG02_(PostgreSQL-아키텍처-및-특징)]]
- [[2026-06-14-PG03_(PostgreSQL-구동-및-종료-pg_ctl)]]
