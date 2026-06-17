# Airflow 동시실행 제어 (parallelism · max_active)

- **카테고리**: #airflow #admin
- **태그**: #airflow #admin #parallelism #concurrency #pool #scheduler
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-airflow 동시실행 제어 전역설정 - BigDataTeam]]

## 1. 핵심 요약

Airflow 동시 실행은 **전역(parallelism, max_active_*_per_dag) → DAG별(max_active_runs, max_active_tasks) → Pool/worker concurrency** 순으로 한도가 걸린다. 태스크가 큐에만 머무를 때 이 순서대로 한도 도달 여부를 점검한다.

---

## 2. 전역 설정 (스케줄러 전체)

| 설정 | 의미 | 환경변수 |
|------|------|----------|
| `parallelism` | 전체 동시 태스크 수(전역 한도) | `AIRFLOW__CORE__PARALLELISM` |
| `max_active_runs_per_dag` | DAG당 활성 DagRun 전역 기본값 | `AIRFLOW__SCHEDULER__MAX_ACTIVE_RUNS_PER_DAG` |
| `max_active_tasks_per_dag` | DAG당 동시 태스크 전역 기본값 | `AIRFLOW__SCHEDULER__MAX_ACTIVE_TASKS_PER_DAG` |

```bash
docker exec -it airflow-airflow-scheduler-1 airflow config get-value core parallelism
docker exec -it airflow-airflow-scheduler-1 airflow config get-value scheduler max_active_runs_per_dag
```

## 3. DAG별 설정 (전역 기본값 덮어씀)

```python
with DAG(
    dag_id="devstg_useract_stat_clickpath_summary",
    max_active_runs=1,
    max_active_tasks=8,   # Airflow 2.10 지원
) as dag:
    ...
```
- UI: DAG 화면 → **Details** 탭에서 확인.

## 4. 왜 큐에만 머무르나? (체크 순서)

1. 전역 **parallelism** 한도 도달
2. 전역 **max_active_*_per_dag** 한도
3. DAG별 **max_active_runs / max_active_tasks**
4. **Pools / worker concurrency** (풀 슬롯, Celery/K8s 워커 동시성)

## 5. 연관 개념

- [[2026-06-14-AF07_(Airflow-Backfill)]] — backfill 시 동시성 한도 영향
- [[2026-06-14-AF02_(Airflow-Docker-Compose-설치)]]
