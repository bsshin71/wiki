# PostgreSQL RPM 설치 가이드 (RHEL/Rocky Linux)
- **카테고리**: #DBMS #PostgreSQL
- **태그**: #PostgreSQL #install #RPM #Rocky-Linux #RHEL
- **작성일**: 2026-06-02
- **참조 원본**: [[2026-06-02-postresql rpm 설치과정]], [[2026-06-02-postresql설치(ver 18)]], [[2026-06-02-Rocky9+PG설치 (rpm)]]

## 1. 핵심 요약
- RHEL/Rocky Linux 환경에서 공식 PostgreSQL YUM 저장소를 사용한 RPM 설치 가이드. PG16(Rocky9)과 PG18(RHEL8) 버전별 절차 포함.

## 2. 상세 설명

### PG18 설치 — RHEL 8 계열 (rpm 설치과정)

```bash
# 1. 기존 충돌 저장소 삭제
rm -f /etc/yum.repos.d/pgdg*.repo
dnf clean all

# 2. 공식 PostgreSQL 저장소 RPM 설치
dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-8-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# 3. RHEL 8 내장 PostgreSQL 모듈 비활성화 (버전 충돌 방지)
dnf -qy module disable postgresql

# 4. PostgreSQL 18 설치
dnf install -y postgresql18-server postgresql18

# 5. DB 초기화 및 서비스 시작
/usr/pgsql-18/bin/postgresql-18-setup initdb
systemctl enable --now postgresql-18
```

**데이터 디렉토리 경로 변경 (선택)**:
```bash
systemctl stop postgresql-18
mkdir -p /home/postgres/pg18/data
cp -a /var/lib/pgsql/18/data/* /home/postgres/pg18/data/
chown -R postgres:postgres /home/postgres/pg18
chmod 700 /home/postgres/pg18/data
chmod 600 /home/postgres/pg18/data/*.conf

# systemd 서비스 환경변수 수정 (systemctl edit postgresql-18)
[Service]
Environment=PGDATA=/home/postgres/pg18/data

# SELinux 보안 컨텍스트 (RHEL 8 필수)
semanage fcontext -a -t postgresql_db_t "/home/postgres/pg18/data(/.*)?"
restorecon -R -v /home/postgres/pg18/data

systemctl daemon-reload
systemctl start postgresql-18
```

**외부 접속 허용**:
```bash
# postgresql.conf
listen_addresses = '*'

# pg_hba.conf
host all all 0.0.0.0/0 scram-sha-256

systemctl restart postgresql-18
```

**postgres 계정 외부 접속 비밀번호 설정**:
```bash
su - postgres -c "psql -c \"ALTER USER postgres PASSWORD '비밀번호';\""
```

---

### PG16 설치 — Rocky Linux 9 (rpm)

CentOS 지원 종료로 Rocky Linux 9에 PostgreSQL 16 설치.

```bash
# 1. 공식 저장소 설치
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# 2. 내장 모듈 비활성화
sudo dnf -qy module disable postgresql

# 3. PG16 서버 설치
sudo dnf install -y postgresql16-server

# 4. 데이터 디렉토리 및 로그 디렉토리 생성
mkdir -p /home/user/postgresql16/data
mkdir -p /home/user/postgresql16/logs

# 5. bin 소프트링크
ln -s /usr/pgsql-16/bin /home/user/postgresql16/bin

# 6. DB 초기화
./postgresql16/bin/initdb -D /home/user/postgresql16/data

# 7. 설정 파일 수정 (postgresql.conf)
log_directory = '/home/user/postgresql16/logs'
log_filename = '%Y-%m-%d.log'
unix_socket_directories = '/home/user/postgresql16/data'

# 8. 서버 시작
./postgresql16/bin/pg_ctl -D /home/user/postgresql16/data start

# 9. 환경변수 (.bash_profile)
export PGHOST=/home/user/postgresql16/data

# 10. 기본 DB 생성 및 접속 확인
./postgresql16/bin/createdb user
psql
```

**방화벽 포트 오픈**:
```bash
sudo firewall-cmd --permanent --zone=public --add-port=5432/tcp
sudo firewall-cmd --reload
```

---

### 오류 대처

| 상황 | 해결 |
|------|------|
| 포그라운드로 시작 필요 | `su - postgres -c "/usr/pgsql-18/bin/postgres -D /home/postgres/pg18/data"` |
| 기존 저장소 충돌 | `rm -f /etc/yum.repos.d/pgdg*.repo && dnf clean all` |

## 3. 연관 개념 (지식 연결)
_(관련 문서가 생성되면 여기에 백링크를 추가합니다.)_
