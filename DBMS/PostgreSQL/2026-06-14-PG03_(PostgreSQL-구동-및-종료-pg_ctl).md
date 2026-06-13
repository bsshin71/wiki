# PostgreSQL 구동 및 종료 (pg_ctl)

- **카테고리**: #DBMS #PostgreSQL #admin
- **태그**: #PostgreSQL #pg_ctl #기동 #종료 #shutdown_mode
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-postgresql 구동 및 종료 - BigDataTeam]]

## 1. 핵심 요약

서버 기동/종료/리로드는 **`pg_ctl`** 로 수행하며, `-D`로 데이터 클러스터 위치를 지정합니다.
종료 모드는 **smart(접속 끊김 대기) / fast(롤백 후 종료, 권장) / immediate(강제, 재기동 시 recovery)** 3가지입니다.

---

## 2. pg_ctl 명령

```bash
pg_ctl start   [-w] [-D DATADIR]
pg_ctl stop    [-w] [-D DATADIR] [-m SHUTDOWN-MODE]
pg_ctl restart [-w] [-D DATADIR] [-m SHUTDOWN-MODE]
pg_ctl reload  [-D DATADIR]
pg_ctl status  [-D DATADIR]
```
- `-D`: 데이터 클러스터(PGDATA) 위치.
- `-w`: 명령이 정상 완료될 때까지 대기.

## 3. Shutdown Mode

| 모드 | 옵션 | 동작 |
|------|------|------|
| smart | `-ms` | 모든 client 접속 종료를 대기 |
| **fast** | `-mf` | 진행 작업 rollback + 접속 끊고 종료 (권장) |
| immediate | `-mi` | 강제 종료, 다음 기동 시 recovery 수행 |

```bash
pg_ctl stop -D /dbdata -mf -w
```

## 4. 연관 개념

- [[2026-06-14-PG01_(PostgreSQL-소스-컴파일-설치)]]
- [[2026-06-14-PG04_(PostgreSQL-데이터베이스-관리)]]
