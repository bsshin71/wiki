# PostgreSQL pgBackRest 백업 가이드
- **카테고리**: #DBMS #PostgreSQL
- **태그**: #PostgreSQL #backup #pgBackRest #운영
- **작성일**: 2026-06-02
- **참조 원본**: [[2026-06-02-pgBackRest 초기화 및 전체 백업 수행 절차]]

## 1. 핵심 요약
- pgBackRest는 PostgreSQL 전용 백업·복구 도구. Stanza 생성 → 설정 검증 → 전체 백업 → 증분 백업 순서로 운영.

## 2. 상세 설명

### 사전 준비

`postgresql.conf`에 pgBackRest 아카이빙 설정이 되어 있어야 함. 변경 후 DB 재시작 필요.

```ini
archive_mode = on
archive_command = 'pgbackrest --stanza=mydb archive-push %p'
```

---

### 실행 절차

**단계 1 — Stanza 생성**

특정 DB 클러스터에 대한 백업 설정 단위. 백업 저장소에 필요한 구조 생성.

```bash
# postgres 계정으로 실행
pgbackrest --stanza=mydb stanza-create
```

**단계 2 — 설정 검증**

PostgreSQL ↔ pgBackRest 통신 및 아카이브 경로 확인.

```bash
pgbackrest --stanza=mydb check
# 성공 시: status: ok
```

> ⚠️ 에러 발생 시 `postgresql.conf`의 `archive_command` 설정 재확인.

**단계 3 — 전체 백업**

```bash
pgbackrest --stanza=mydb backup --type=full
```

- 처음 실행 시 반드시 `--type=full` 지정.
- 진행 상황 로그에서 완료 확인.

**단계 4 — 백업 결과 확인**

```bash
pgbackrest info
```

---

### 관리·모니터링

**증분 백업** (전체 백업 이후):
```bash
pgbackrest --stanza=mydb backup --type=incr
```

**crontab 자동화 예시**:
```cron
# 매주 일요일 전체 백업
0 1 * * 0 pgbackrest --stanza=mydb backup --type=full

# 매일 자정 증분 백업
0 0 * * 1-6 pgbackrest --stanza=mydb backup --type=incr
```

**보관 정책**: `repo1-retention-full=2` → 최신 전체 백업 2개까지 유지, 나머지 자동 삭제.

---

### 주의사항

```bash
# pg_tblspc 링크가 PGDATA 외부를 가리키는지 확인 (필수)
ls -al /home/postgres/pg18/data/pg_tblspc
# 결과: .../data/pg_tblspc/... -> /home/postgres/pg18/ts_data/...
```

> Tablespace 링크가 PGDATA 안을 가리키면 `[070] ERROR` 발생.

복구(Restore) 테스트까지 주기적으로 수행하는 것을 권장.

## 3. 연관 개념 (지식 연결)
- [[2026-06-02_(PostgreSQL-RPM-설치-가이드)]] — PostgreSQL 설치 (pgBackRest 설치 환경)
