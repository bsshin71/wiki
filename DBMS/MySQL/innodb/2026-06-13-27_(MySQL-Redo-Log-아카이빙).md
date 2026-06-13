# MySQL Redo Log 아카이빙

- **카테고리**: #DBMS #MySQL #InnoDB
- **태그**: #InnoDB #redo_log #아카이빙 #백업
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-리두로그 아카이빙 - BigDataTeam]]

## 1. 핵심 요약

MySQL 8.0부터 redo log를 별도 경로에 아카이빙하는 기능이 추가되었습니다.
백업 도중 발생한 변경을 누락 없이 확보하기 위해 사용하며, **아카이브 디렉터리는 chmod 700 필수**, 시작 세션 연결이 끊기면 아카이빙도 중단됩니다.

---

## 2. 설정 과정

### 2.1 아카이브 경로 생성 (권한 700 필수)

```bash
mkdir archlogs
chmod 700 archlogs      # 필수: 다른 OS 유저 접근 가능하면 ERROR 3846
```

### 2.2 시스템 변수 설정

```sql
SET GLOBAL innodb_redo_log_archive_dirs='backup:/home/bos2/mysql/archlogs';
```

### 2.3 아카이빙 스레드 시작/중지

```sql
-- 시작 (label:backup)
DO innodb_redo_log_archive_start('backup');
-- chmod 700 안 했으면: ERROR 3846 (HY000): Redo log archive directory ... is accessible to all OS users

-- 중지
DO innodb_redo_log_archive_stop();
```

---

## 3. 동작 확인

```sql
-- 대량 로그 생성으로 아카이빙 유도
CREATE TABLE test( id BIGINT AUTO_INCREMENT, data MEDIUMTEXT, PRIMARY KEY(id) );
INSERT INTO test(data) SELECT REPEAT('123456789',10000) FROM employees.salaries LIMIT 100;
COMMIT;
```

```bash
# 아카이브 로그 파일 생성 확인
ls -al archlogs
# -r--r-----. 1 bos2 bos2 9691136 ... archive.b4ee5afc-...000001.log
```

---

## 4. 주의사항

- 로그 파일 스위치 시 복사되는 것이 아니라, **redo log 엔트리가 추가될 때 함께 기록**되는 방식.
- 데이터 변경이 발생하면 아카이빙 로그 파일 크기가 조금씩 증가.
- ⛔ **`innodb_redo_log_archive_start`를 실행한 세션의 연결이 끊기면 아카이빙도 중단**된다.

---

## 5. 연관 개념

- [[2026-06-13-26_(MySQL-Redo-Log-및-로그버퍼-설정)]]
- [[2026-06-13-28_(MySQL-Redo-Log-활성화-비활성화)]]
- [[2026-06-13_(MySQL-XtraBackup-백업복구-가이드)]]
