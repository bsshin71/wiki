# Airflow DB Clean (메타DB 정리)

- **카테고리**: #airflow #admin
- **태그**: #airflow #admin #db-clean #metadb #maintenance
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-airflow db clean - BigDataTeam]]

## 1. 핵심 요약

Airflow는 실행 결과 정보를 메타DB에 계속 쌓으므로, `airflow db clean --clean-before-timestamp`로 **특정 시점 이전 이력을 주기적으로 삭제**해야 한다. 실삭제 전 `--dry-run`으로 대상을 확인한다.

---

## 2. 삭제 절차

```bash
cd /home/bos/airflow

# 1) 삭제 대상 확인 (dry-run)
./airflow.sh db clean --clean-before-timestamp '2025-03-01 00:00:00' --dry-run

# 2) 실제 삭제 (-y: 확인 생략)
./airflow.sh db clean --clean-before-timestamp '2025-03-01 00:00:00' -y
```

| 옵션 | 의미 |
|------|------|
| `--clean-before-timestamp` | 이 시각 **이전** 이력 삭제 |
| `--dry-run` | 삭제하지 않고 대상만 표시 |
| `-y` | 확인 프롬프트 생략 |

> 정기 스케줄(예: cron/별도 DAG)로 운영하면 메타DB 비대화를 예방할 수 있다.

## 3. 연관 개념

- [[2026-06-14-AF02_(Airflow-Docker-Compose-설치)]]
- [[2026-06-14-AF07_(Airflow-Backfill)]]
