# MySQL XtraBackup 설치 및 백업 계정 권한

- **카테고리**: #DBMS #MySQL #Backup
- **태그**: #backup #XtraBackup #설치 #권한
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-Xtrabackup Install setting - BigDataTeam]]

## 1. 핵심 요약

Percona XtraBackup은 percona-release 저장소로 설치하며, **버전별로 백업 계정에 요구되는 권한이 다릅니다**.
8.0.35+에서는 `FLUSH_*` 권한과 옵션명 변경(`--compress-threads`)에 주의하고, 인증 플러그인 오류 시 `mysql_native_password`로 전환합니다.

---

## 2. 설치

```bash
# repository 설치
sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm

# repository 활성화
sudo percona-release enable-only tools release

# percona-xtrabackup-80 설치
sudo yum install percona-xtrabackup-80
```

**특징**: 무중단 InnoDB Hot Backup, 증분 백업, 압축 스트리밍, 온라인 테이블 이동, slave 생성 용이, 서버 부하 최소.

---

## 3. 백업 계정 권한 (버전별)

### 3.1 권한 의미

| 권한 | 용도 |
|------|------|
| RELOAD, LOCK TABLES | FLUSH TABLES WITH READ LOCK, FLUSH ENGINE LOGS |
| BACKUP_ADMIN | performance_schema.log_status 조회, LOCK INSTANCE/BINLOG/TABLES FOR BACKUP |
| REPLICATION CLIENT | binary log position 획득 |
| PROCESS | SHOW ENGINE INNODB STATUS |
| CREATE/INSERT/SELECT | PERCONA_SCHEMA.xtrabackup_history 관리 |

### 3.2 XtraBackup 2.4 이하

```sql
CREATE USER 'bkpuser'@'localhost' IDENTIFIED BY 'Admin@1234';
GRANT PROCESS, RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'bkpuser'@'localhost';
GRANT SELECT ON performance_schema.log_status TO 'bkpuser'@'localhost';
FLUSH PRIVILEGES;
```

### 3.3 XtraBackup 8.0 ~ 8.0.20

```sql
GRANT BACKUP_ADMIN, PROCESS, RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'bkpuser'@'localhost';
GRANT SELECT ON performance_schema.log_status TO 'bkpuser'@'localhost';
GRANT SELECT ON performance_schema.keyring_component_status TO bkpuser@'localhost';
```

### 3.4 XtraBackup 8.0.35 ~

```sql
GRANT BACKUP_ADMIN, PROCESS, RELOAD, LOCK TABLES, REPLICATION CLIENT,
      FLUSH_OPTIMIZER_COSTS, FLUSH_STATUS, FLUSH_TABLES, FLUSH_USER_RESOURCES ON *.* TO 'bkpuser'@'localhost';
GRANT SELECT ON performance_schema.log_status TO 'bkpuser'@'localhost';
GRANT SELECT ON performance_schema.keyring_component_status TO bkpuser@'localhost';
GRANT SELECT ON performance_schema.replication_group_members TO bkpuser@'localhost';
```

---

## 4. 주의사항

- **8.0.35+ 옵션명 변경**: `--compress-thread` → `--compress-threads` (구 변수명 사용 시 오류).
- **인증 플러그인 오류**(caching_sha2_password) 시:
  ```sql
  ALTER USER 'bkpuser'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Admin@1234';
  ```

---

## 5. 연관 개념

- [[2026-06-13-35_(MySQL-XtraBackup-동작원리)]]
- [[2026-06-13-36_(MySQL-XtraBackup-백업-요소-및-옵션)]]
- [[2026-06-13_(MySQL-Percona-설치-가이드)]]
