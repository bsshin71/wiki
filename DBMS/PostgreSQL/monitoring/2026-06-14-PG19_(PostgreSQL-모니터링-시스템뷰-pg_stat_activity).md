# PostgreSQL 모니터링 시스템 뷰 (pg_stat_activity)

- **카테고리**: #DBMS #PostgreSQL #admin
- **태그**: #PostgreSQL #모니터링 #pg_stat_activity #wait_event #backend_type
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-모니터링 주요 시스템 뷰, 함수 - BigDataTeam]]

## 1. 핵심 요약

`pg_stat_activity`는 **1행=1서버 프로세스**로, 세션의 state·wait_event·query·backend_type을 보여주는 핵심 모니터링 뷰입니다.
`state`(active/idle/idle in transaction…)와 `wait_event_type`(Lock/IO/Client…)으로 세션이 무엇을 기다리는지 진단합니다.

---

## 2. 주요 컬럼

| 컬럼 | 의미 |
|------|------|
| datname / pid / usename | DB명 / 프로세스 ID / 로그인 사용자 |
| client_addr / application_name | 클라이언트 IP / 앱 이름 |
| backend_start / xact_start / query_start | 프로세스·트랜잭션·쿼리 시작 시각 |
| state_change | state 마지막 변경 시각 |
| wait_event_type / wait_event | 대기 이벤트 유형/이름 |
| state | 세션 상태 |
| backend_xid / backend_xmin | 트랜잭션 ID / xmin horizon |
| query / query_id | 현재(또는 마지막) 쿼리 |
| backend_type | 백엔드 종류 |

## 3. state 값

| state | 의미 |
|-------|------|
| `active` | 쿼리 실행 중 |
| `idle` | 새 명령 대기 |
| `idle in transaction` | 트랜잭션 중 쿼리 미실행 (⚠️ 장기 시 락 점유) |
| `idle in transaction (aborted)` | 트랜잭션 중 에러 발생 상태 |
| `fastpath function call` | fast-path 함수 실행 |
| `disabled` | track_activities 비활성 |

## 4. wait_event_type

`Activity, BufferPin, Client, Extension, IO, IPC, Lock, LWLock, Timeout`
> `wait_event_type=Lock`이면 잠금 대기 → 블로킹 분석 필요.

## 5. backend_type

`client backend, autovacuum launcher/worker, logical replication launcher/worker, parallel worker, background writer, checkpointer, archiver, walreceiver, walsender, walwriter, startup` 등.

## 6. 예시 해석

```
state            | idle in transaction
wait_event_type  | Client / wait_event | ClientRead
query            | create table test (id int);
```
> 트랜잭션을 열어둔 채 클라이언트 입력 대기(idle in transaction) → 장기화 시 락/vacuum 방해.

## 7. 연관 개념

- [[2026-06-14-PG20_(PostgreSQL-상황별-모니터링-쿼리)]]
- [[2026-06-14-PG08_(PostgreSQL-세션-관리-연결제어)]]
- [[2026-06-02_(PostgreSQL-모니터링-쿼리-모음)]]
