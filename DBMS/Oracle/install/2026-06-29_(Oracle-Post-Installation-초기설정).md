# Oracle Post Installation 초기 설정
- **카테고리**: #Oracle #install
- **태그**: #Oracle #install #FILESYSTEMIO #DBMS_SCHEDULER #PFILE
- **작성일**: 2026-06-29
- **참조 원본**: [[2026-06-20_oracle post installation]]
- **is_public**: true

## 목차
- [[#1. 핵심 요약]]
- [[#2. FILESYSTEMIO 설정 (Direct & Async I/O)]]
- [[#3. DBMS_SCHEDULER 비활성화]]
- [[#4. PFILE 생성]]
- [[#5. 연관 개념]]

## 1. 핵심 요약
Oracle DB 생성 직후 권장하는 초기화 작업 3가지. ① 파일 I/O를 Direct+Async로 전환(`FILESYSTEMIO_OPTIONS=SETALL`) — I/O 성능 향상. ② 자동 유지보수 스케줄러 비활성화(`DBMS_AUTO_TASK_ADMIN.DISABLE`). ③ 현재 SPFILE을 편집 가능한 PFILE로 백업 생성(`CREATE PFILE FROM SPFILE`).

## 2. FILESYSTEMIO 설정 (Direct & Async I/O)

DB 생성 후 파일 I/O를 Direct + Async 방식으로 설정한다.

```sql
-- 현재 설정 확인
SQL> show parameter FILESYSTEMIO;
-- filesystemio_options    string    none

-- SETALL: Direct I/O + Async I/O 모두 활성화 (재기동 필요)
ALTER SYSTEM SET FILESYSTEMIO_OPTIONS=SETALL SCOPE=SPFILE;

-- 재기동 후 적용 확인
SQL> show parameter FILESYSTEMIO;
-- filesystemio_options    string    SETALL
```

## 3. DBMS_SCHEDULER 비활성화

Oracle 자동 유지보수 작업(통계 수집, 세그먼트 어드바이저 등)을 비활성화한다.

```sql
EXECUTE DBMS_AUTO_TASK_ADMIN.DISABLE;
```

## 4. PFILE 생성

운영 중인 SPFILE을 사람이 편집 가능한 텍스트 PFILE로 백업 생성한다.

```sql
SQL> CREATE PFILE FROM SPFILE;
```

생성 결과 (`$ORACLE_HOME/dbs/`):

```
initdevdb.ora    ← PFILE (텍스트, 신규 생성)
spfiledevdb.ora  ← SPFILE (바이너리, 원본 유지)
```

## 5. 연관 개념
- [[2026-06-05_(Oracle-19c-Rocky-Linux-설치-가이드)]]
- [[2026-06-05_(Oracle-DBCA-NETCA-가이드)]]
