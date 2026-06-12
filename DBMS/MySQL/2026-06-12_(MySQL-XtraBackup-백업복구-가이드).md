# MySQL XtraBackup 백업·복구 가이드
- **카테고리**: #DBMS #MySQL
- **태그**: #MySQL #backup #XtraBackup #Percona #시점복구 #장애복구
- **작성일**: 2026-06-12
- **참조 원본**: [[2026-06-12-Xtrabackup Install setting - BigDataTeam]], [[2026-06-12-XtraBackup의 동작원리 - BigDataTeam]], [[2026-06-12-Xtrabackup 백업 요소 - BigDataTeam]], [[2026-06-12-Full 백업 및 복구 - BigDataTeam]], [[2026-06-12-증분 백업 - BigDataTeam]], [[2026-06-12-증분 백업 복구 테스트 - BigDataTeam]], [[2026-06-12-증분백업script(V8) - BigDataTeam]], [[2026-06-12-full백업script ( V8 ) - BigDataTeam]], [[2026-06-12-full백업script( v2.4 구버전용) - BigDataTeam]], [[2026-06-12-백업 복구 시나리오 - BigDataTeam]], [[2026-06-12-백업 복구 테스트 - BigDataTeam]], [[2026-06-12-시점 복구 - BigDataTeam]], [[2026-06-12-시점 복구 테스트 - BigDataTeam]], [[2026-06-12-긴급장애 복구 - BigDataTeam]]

## 1. 핵심 요약
- XtraBackup은 InnoDB **Hot Backup**(운영 중단 없이) 지원. Full → Incremental → 시점복구 순으로 복구 전략 구성.
- 증분 백업 복구 시 마지막 증분 전에는 `--redo-only` 필수. 마지막 증분에서만 rollback 적용.
- 긴급장애 시 `innodb_force_recovery=1~6` 단계적 시도 후 mysqldump로 데이터 추출.

## 2. 상세 설명

### XtraBackup 설치 및 계정 설정

```bash
# 설치
sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
sudo percona-release enable-only tools release
sudo yum install percona-xtrabackup-80
```

**백업 계정 생성 (버전별 권한 상이)**:

```sql
-- XtraBackup 8.0.35+
CREATE USER 'bkpuser'@'localhost' IDENTIFIED BY '****';
GRANT BACKUP_ADMIN, PROCESS, RELOAD, LOCK TABLES, REPLICATION CLIENT,
      FLUSH_OPTIMIZER_COSTS, FLUSH_STATUS, FLUSH_TABLES, FLUSH_USER_RESOURCES
      ON *.* TO 'bkpuser'@'localhost';
GRANT SELECT ON performance_schema.log_status TO 'bkpuser'@'localhost';
GRANT SELECT ON performance_schema.keyring_component_status TO bkpuser@'localhost';
GRANT SELECT ON performance_schema.replication_group_members TO bkpuser@'localhost';
FLUSH PRIVILEGES;
```

> ⚠️ 버전별 필요 권한이 다름. 버전 업그레이드 시 권한 재확인 필요.

---

### Full 백업 및 복구

**백업**:
```bash
innobackupex --defaults-file=/etc/my.cnf \
  --user=bkpuser --password=*** \
  /backup/mysql/fullbackup

# 결과 파일
# xtrabackup_binlog_info  : 백업 완료 시점 Binlog 파일·포지션
# xtrabackup_checkpoints  : 백업 종류(full/incremental) + LSN
# xtrabackup_logfile      : 백업 중 변경된 블록 (복구에 필수)
```

**복구 절차**:
```bash
# 1. 복구 준비 (apply-log)
innobackupex --defaults-file=/etc/my.cnf \
  --apply-log /backup/fullbackup/2019-06-30_04-08-09

# 2. MySQL 종료 + 기존 데이터 백업
systemctl stop mysqld
mv /var/lib/mysql /var/lib/mysql-bak
mkdir /var/lib/mysql

# 3. 백업 파일 복사
innobackupex --copy-back /backup/fullbackup/2019-06-30_04-08-09
chown -R mysql:mysql /var/lib/mysql

# 4. MySQL 시작
systemctl start mysqld
```

---

### 증분 백업 및 복구

**백업 시나리오 (Full → Inc1 → Inc2)**:
```bash
# Full 백업 (일요일)
innobackupex --user=bkpuser --password=*** /backup/fullbak

# 1차 증분 (월요일) - 이전 백업 경로 기준
innobackupex --user=bkpuser --password=*** \
  --incremental --incremental-basedir=/backup/fullbak/2019-06-30_10-02-36 \
  /backup/inc1

# 2차 증분 (화요일)
innobackupex --user=bkpuser --password=*** \
  --incremental --incremental-basedir=/backup/inc1/2019-06-30_10-39-41 \
  /backup/inc2
```

**복구 절차 (LSN 순서로 적용)**:
```bash
# 1. Full 백업 redo-only 적용
innobackupex --apply-log --redo-only /backup/fullbak/날짜

# 2. 1차 증분 redo-only 적용 (마지막 증분 전까지 redo-only 필수)
innobackupex --apply-log --redo-only /backup/fullbak/날짜 \
  --incremental-dir=/backup/inc1/날짜

# 3. 마지막 증분 적용 (--redo-only 제거 → rollback 포함)
innobackupex --apply-log /backup/fullbak/날짜 \
  --incremental-dir=/backup/inc2/날짜

# 4. Full 백업 파일 최종 apply-log
innobackupex --apply-log /backup/fullbak/날짜

# 5~8. MySQL 종료 → 기존 데이터 이동 → copy-back → 시작 (Full 복구와 동일)
```

---

### 시점 복구 (Point-in-Time Recovery)

백업 완료 시점 이후 특정 시점까지 복구:

```bash
# 1. Full 백업으로 복구 완료
# 2. 백업 시점의 Binlog position 확인
cat /backup/fullbak/날짜/xtrabackup_binlog_info
# bin_log.000001  154

# 3. mysqlbinlog으로 변경분 SQL 변환
mysqlbinlog bin_log.000001 --start-position=154 > 001.sql
mysqlbinlog bin_log.000002 > 002.sql
mysqlbinlog bin_log.000003 --stop-position=8564 > 003.sql  # 원하는 시점

# 4. 순차적으로 SQL 적용
mysql -uroot -p < 001.sql
mysql -uroot -p < 002.sql
mysql -uroot -p < 003.sql
```

**mysqlbinlog 주요 옵션**:

| 옵션 | 설명 |
|------|------|
| `--start-position=N` | N 이상의 포지션부터 읽기 시작 |
| `--stop-position=N` | N 직전까지만 읽기 |
| `--start-datetime=DATETIME` | 특정 시각부터 시작 |
| `--stop-datetime=DATETIME` | 특정 시각까지만 읽기 |
| `--database=DB명` | 특정 DB 관련 이벤트만 |

---

### 긴급 장애 복구 (innodb_force_recovery)

MySQL 자동 복구 실패 시 강제 시작:

```ini
# my.cnf에 설정
[mysqld]
innodb_force_recovery=1
```

| 값 | 상황 | 조치 |
|----|------|------|
| 1 | 데이터/인덱스 페이지 손상 | mysqldump 후 DB 재구축 |
| 2 | undo 레코드 삭제 중 장애 | |
| 3 | 미커밋 트랜잭션 롤백 안 함 | mysqldump 후 재구축 |
| 4 | Insert Buffer 손상 | |
| 5 | Undo 로그 사용 불가, 미커밋 TX도 커밋 처리됨 | mysqldump 후 재구축 |
| 6 | Redo 로그 손상, 마지막 체크포인트 시점만 복구 | Redo 로그 삭제 후 재시작 |

> ⚠️ 값이 커질수록 데이터 손실 가능성 증가. 최소값부터 시작.

**오류 대처**:
```bash
# 권한 오류 (selinux)
setenforce 0
systemctl start mysqld

# socket 오류
systemctl stop mysqld
chmod 777 /var/lib/mysql
systemctl start mysqld
```

## 3. 연관 개념 (지식 연결)
- [[2026-06-12_(MySQL-Replication-가이드)]] — XtraBackup을 이용한 Slave 추가
- [[2026-06-12_(MySQL-Percona-설치-가이드)]] — XtraBackup 설치 환경 (Percona Server)
