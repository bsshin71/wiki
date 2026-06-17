# MySQL pt-table-sync — 테이블 데이터 동기화

- **카테고리**: #DBMS #MySQL #tools
- **태그**: #MySQL #tools #pt-table-sync #percona-toolkit #replication #data-sync
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-pt-table-sync 사용하기 - BigDataTeam]]

## 1. 핵심 요약

`pt-table-sync`는 **MySQL 테이블 간 데이터 불일치를 찾아 동기화**하는 Percona Toolkit 도구. `--sync-to-master`로 slave를 master에 맞추거나, DSN을 여러 개 지정해 소스→대상 방향으로 sync한다. 사전에 `pt-table-checksum`으로 불일치를 확인하고, **PK 없는 테이블은 sync 불가**.

---

## 2. 기본 사용 절차

```bash
# Step 1: 불일치 확인 (체크섬 비교)
pt-table-checksum h=db1,u=root,p='pass' --tables='t4' --no-check-binlog-format

# Step 2: slave 특정 테이블을 master에 맞게 sync
pt-table-sync --execute --sync-to-master h=db2,u=root,p='pass',D=sourcedb,t=t4

# Step 3: 전체 테이블 sync (주의: PK 없는 테이블 오류 발생 가능)
pt-table-sync --execute --sync-to-master h=db2,u=root,p='pass'
```

## 3. 주요 옵션

| 옵션 | 의미 |
|------|------|
| `--sync-to-master` | DSN이 slave → master에 맞춰 동기화 |
| `--execute` | 실제 변경 실행 (없으면 dry-run) |
| `--replicate` | 복제 채널을 통한 sync |
| `h=`, `D=`, `t=` | 호스트·DB·테이블 지정 |

## 4. 주의사항

- **PK 없는 테이블**: `Can't make changes on the master because no unique index exists` 오류
- **generated column 포함 테이블**(mysql.engine_cost 등): sync 오류 발생 가능 → 테이블 지정 제외 권장
- **트리거 있는 테이블**: `Triggers are defined on the table` 오류 → `--ignore-tables` 또는 `--no-check-triggers`

## 5. 연관 개념

- [[2026-06-14-MS40_(MySQL-Percona-Toolkit-도구-모음)]]
- [[2026-06-14-MS14_(MySQL-pt-online-schema-change-pt-osc)]] — pt-osc (같은 Percona Toolkit)
- [[2026-06-13-06_(MySQL-Replication-에러-처리-및-복구)]]
