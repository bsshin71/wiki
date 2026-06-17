# Airflow Backfill (과거 구간 재처리)

- **카테고리**: #airflow #admin
- **태그**: #airflow #admin #backfill #dagrun #rerun
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-airflow backfill - BigDataTeam]]

## 1. 핵심 요약

Backfill은 **특정 시점의 DAG/Task를 재실행**하는 것. `airflow dags backfill <dag> --reset-dagruns`(전체 재처리) 또는 `--rerun-failed-tasks`(실패분만)로 수행하며, **scheduler 컨테이너**에서 실행하는 것이 자연스럽다. parallelism·max_active_runs·Pool 한도 때문에 한 번에 많이 못 돌 수 있어 범위를 쪼개 시운전 후 확대한다.

---

## 2. 기본 실행

```bash
docker exec -it airflow-airflow-scheduler-1 bash -lc \
  "airflow dags backfill devstg_useract_stat_clickpath_summary --reset-dagruns"
```

## 3. 단계별 실행

```bash
# 0) (선택) 스케줄러가 새 런 안 만들게 일시중지
airflow dags pause <dag>
# 1) 백필
airflow dags backfill <dag> --reset-dagruns
# 2) 진행 확인
airflow dags list-runs -d <dag> --no-backfill
# 3) (선택) 재개
airflow dags unpause <dag>
```

## 4. 패턴별 옵션

| 목적 | 옵션 |
|------|------|
| 전체 재처리(성공분 포함) | `--reset-dagruns` |
| 실패분만 재처리 | `--rerun-failed-tasks` |
| 특정 태스크만 전 구간 | `-t <task_id> --reset-dagruns` |

## 5. 주의 사항

- **실행 위치**: `scheduler` 컨테이너 권장 (런 생성·스케줄링 담당).
- **따옴표**: ISO 시각에 `:` 포함 → `-s/-e` 값은 따옴표로 감쌀 것.
- **동시성 한도**: `max_active_runs`, `parallelism`, Pool 제한으로 한 번에 많이 못 돎 → 범위 분할.
- **Dynamic Task Mapping**: 일부 인덱스 실패 시 DagRun 실패로 남음 → 로그 확인.
- **테스트 권장**: 짧은 범위(2~3구간) 시운전 후 전체 확대.

## 6. 연관 개념

- [[2026-06-14-AF05_(Airflow-동시실행-제어-parallelism)]] — backfill 동시성 한도
- [[2026-06-14-AF06_(Airflow-DB-Clean)]]
