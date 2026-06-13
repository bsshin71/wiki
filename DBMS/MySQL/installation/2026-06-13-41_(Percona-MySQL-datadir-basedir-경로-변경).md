# Percona MySQL datadir·basedir 경로 변경

- **카테고리**: #DBMS #MySQL #Installation
- **태그**: #install #datadir #basedir #systemd
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-Percona MySQL 경로 수정 - BigDataTeam]]

## 1. 핵심 요약

YUM으로 설치한 Percona MySQL 8.0의 데이터·기본 경로를 기본값(`/var/lib/mysql`)이 아닌 별도 경로로 변경하는 절차입니다.
**my.cnf의 datadir/basedir 수정 → datadir 초기화 → systemd 서비스가 변경된 my.cnf를 참조하도록 `mysqld_pre_systemd` 수정**이 핵심입니다.

---

## 2. 전제

- "install percona by rpm on redhat"의 **DB 초기화 직전(9단계)까지** 진행한 상태.
- 예: `datadir=/data/mysql`, `basedir=/opt/percona` 로 변경 가정.

---

## 3. 새 디렉토리 생성 및 권한

```bash
mkdir -p /data/mysql /opt/percona
chown -R mysql:mysql /data/mysql /opt/percona
chmod 750 /data/mysql
```

---

## 4. my.cnf 수정

`/home/mysql/mysql_home/etc/my.cnf`:

```ini
[mysqld]
datadir=/data/mysql
basedir=/opt/percona
socket=/data/mysql/mysql.sock
log-error=/data/mysql/mysqld.log
pid-file=/data/mysql/mysqld.pid
```

---

## 5. datadir 초기화

```bash
# 기존 datadir 백업
mv /var/lib/mysql /var/lib/mysql.bak

# mysql 계정으로 초기화
mysqld --defaults-file=/home/mysql/mysql_home/etc/my.cnf \
  --initialize-insecure --user=mysql
```

---

## 6. systemd 서비스가 my.cnf를 참조하도록 수정 (⭐ 핵심)

### 6.1 환경변수 추가 — `/etc/sysconfig/mysql`

```bash
DEFAULTFILE="--defaults-file=/home/mysql/mysql_home/etc/my.cnf"
MYSQLD_OPTS="--defaults-file=/home/mysql/mysql_home/etc/my.cnf"
```

### 6.2 서비스 유닛 — `mysqld.service`

```ini
ExecStart=/usr/sbin/mysqld $MYSQLD_OPTS
```

### 6.3 `/usr/bin/mysqld_pre_systemd` 수정

```bash
get_option () {
  ...
  [ -e /etc/sysconfig/mysql ] && . /etc/sysconfig/mysql        # ← 라인 추가
  ret=$(/usr/bin/my_print_defaults ${DEFAULTFILE} ... )        # ← ${DEFAULTFILE} 추가
  ...
}
```

> `mysqld_pre_systemd`는 초기화·시작 전 datadir을 검사하므로, 변경된 my.cnf를 참조하지 않으면 기본 경로(`/var/lib/mysql`)를 바라봐 시작이 실패한다.

---

## 7. 서비스 반영

```bash
systemctl daemon-reload
systemctl start mysql
systemctl enable mysql
```

---

## 8. 연관 개념

- [[2026-06-13-14_(MySQL-Percona-설치-상세-가이드)]]
- [[2026-06-13-43_(Percona-MySQL-운영-표준-my.cnf)]]
