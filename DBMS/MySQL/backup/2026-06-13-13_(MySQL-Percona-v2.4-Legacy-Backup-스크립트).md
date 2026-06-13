# MySQL Percona v2.4 Legacy Backup 스크립트

- **카테고리**: #DBMS #MySQL #Backup
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-full백업script( v2.4 구버전용) - BigDataTeam.md]]

## 1. 핵심 요약

Percona XtraBackup 2.4는 **innobackupex 명령**을 사용하는 레거시 버전입니다.
MySQL 5.6/5.7 환경에서 여전히 많이 사용되며, RELOAD 권한만으로 가능하고, --apply-log로 복구 준비를 합니다.
V8과 달리 innobackupex가 deprecated되지 않았으므로 하위호환성이 유지됩니다.

---

## 2. v2.4 사용자 생성

```sql
CREATE USER 'backupuser'@'localhost' IDENTIFIED BY 'password';
GRANT RELOAD, LOCK TABLES, PROCESS, REPLICATION CLIENT ON *.* 
  TO 'backupuser'@'localhost';
FLUSH PRIVILEGES;
```

**권한 특징:**
- BACKUP_ADMIN 불필요
- RELOAD만으로 충분
- MyISAM 테이블 동시성 주의

---

## 3. Full Backup 스크립트

```bash
#!/bin/bash

cat fullbak.sh
innobackupex --defaults-file=/etc/my.cnf \
  --no-lock \
  --user=backupuser \
  --password=password \
  /root/mysqlxtra/fullbak
```

**주요 옵션:**
- `--no-lock`: MyISAM 테이블 일관성 주의
- 경로: /root/mysqlxtra/fullbak

**⚠️ 주의 사항:**
- `--no-lock` 사용 시 MyISAM 데이터 불일치 가능성
- 문제 분석: xtrabackup_logfile에서 LSN 추적

---

## 4. 테스트용 테이블 생성

```sql
CREATE DATABASE baktest;
USE baktest;

CREATE TABLE `bktest` (
  `no` INT(10) NOT NULL AUTO_INCREMENT,
  `value` VARCHAR(20) COLLATE utf8_bin DEFAULT NULL,
  PRIMARY KEY(`no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;

INSERT INTO bktest (value) VALUES ('xxxxxxxxxxxxxx');
INSERT INTO bktest (value) SELECT value FROM bktest LIMIT 100;
```

---

## 5. 백업 파일 구조

```
2019-06-30_04-08-09/
├── ibdata1                          (InnoDB shared tablespace)
├── baktest/                         (데이터베이스 디렉토리)
├── mysql/                           (System tables)
├── xtrabackup_binlog_info          (Binlog 위치정보)
├── xtrabackup_checkpoints          (LSN, 백업타입)
├── xtrabackup_info                 (메타정보)
└── xtrabackup_logfile              (변경된 트랜잭션)
```

---

## 6. 복구 절차

### 6.1 Prepare (--apply-log)

```bash
innobackupex --defaults-file=/etc/my.cnf \
  --apply-log \
  /root/mysqlxtra/2019-06-30_04-08-09
```

### 6.2 MySQL 종료 및 기존 데이터 백업

```bash
systemctl stop mysqld.service

# 기존 데이터 백업
cp -r /var/lib/mysql /backup/old_mysql_backup
rm -rf /var/lib/mysql
mkdir -p /var/lib/mysql
```

### 6.3 Copy-back (복원)

```bash
innobackupex --defaults-file=/etc/my.cnf \
  --copy-back \
  /root/mysqlxtra/2019-06-30_04-08-09

# 권한 설정 (필수)
chown -R mysql:mysql /var/lib/mysql
```

### 6.4 MySQL 시작 및 검증

```bash
systemctl start mysqld.service
mysql -uroot -p -e "SELECT VERSION();"
```

---

## 7. v2.4 특성

| 항목 | 설명 |
|------|------|
| **명령어** | innobackupex (xtrabackup 래퍼) |
| **권한** | RELOAD, LOCK TABLES, PROCESS |
| **지원 버전** | MySQL 5.6, 5.7 |
| **Prepare** | --apply-log |
| **로그 옵션** | --compress-thread (단수형) |
| **호환성** | 하위호환 유지 |

---

## 8. 에러 처리

### 8.1 권한 오류

```
ERROR] Access denied for user 'backupuser'@'localhost'
```

**해결:**
```sql
GRANT RELOAD, LOCK TABLES, PROCESS, REPLICATION CLIENT ON *.* 
  TO 'backupuser'@'localhost';
FLUSH PRIVILEGES;
```

### 8.2 SELinux 오류

```
ERROR] InnoDB: os_file_get_status() failed on './ibdata1'
```

**해결:**
```bash
setenforce 0  # 임시 비활성화
# 또는 정책 수정
```

---

## 9. 연관 개념

- [[2026-06-13-11_(MySQL-Full-Backup-복구-innobackupex)]]
- [[2026-06-13-12_(MySQL-Percona-V8-Full-Backup-스크립트)]]
