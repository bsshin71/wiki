# Airflow DB 백업 완료확인 DAG (멀티 인스턴스)

- **카테고리**: #airflow #usecase
- **태그**: #airflow #usecase #dag #backup #monitoring #SSHOperator #dynamic-task
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-DB backup 완료 여부 확인, rmoldbackup 완료 여부 확인 - BigDataTeam]]

## 1. 핵심 요약

여러 DB 인스턴스의 **backup / rmoldbackup(오래된 백업 삭제) 로그를 동적으로 생성한 SSHOperator 배열로 점검**하고, 전부 성공/일부 실패/전부 실패를 분류해 Telegram으로 통합 알림. `instances` 배열을 순회하며 task를 생성하므로 인스턴스 추가 시 배열만 수정하면 된다.

---

## 2. 패턴 — 인스턴스 배열 → 동적 task 생성

```python
instances = ["dmdb1", "dbosdb1", "dcrmdb1", "dcfddb1", "detcdb1"]

ssh_tasks = []
for instance in instances:
    task = SSHOperator(
        task_id=f"check_backup_{instance}", ssh_conn_id=instance,
        command="""
        cd /home/mysql/dbbackup/trclog;
        today=$(date +%Y%m%d);
        filename="backup_${today}.log"
        test -f "$filename" && echo "success" || echo "fail"
        """,
        cmd_timeout=10, do_xcom_push=True)
    ssh_tasks.append(task)

ssh_tasks >> check_if_success >> send_message   # 배열 >> task 의존성
```

## 3. 결과 분류 (성공/실패 집계)

```python
success = [t.replace("check_backup_","") for t,r in task_results.items() if r=="success"]
fail    = [t.replace("check_backup_","") for t,r in task_results.items() if r=="fail"]
# len(success)==len(instances) → 전체 성공 / len(fail)==len(instances) → 전체 실패 / else 부분 성공
```

| DAG | 점검 대상 | schedule |
|-----|-----------|----------|
| `check_dev_dbs_backup` | `backup_YYYYMMDD.log` 존재 | `0 12 * * *` |
| `check_dev_dbs_rmoldbackup` | `rmoldbackup.log` 오늘자 시작 로그 | `40 12 * * *` |

## 4. 연관 개념

- [[2026-06-14-AF11_(Airflow-파티션-완료확인-알림-DAG)]]
- [[2026-06-14-AF13_(Airflow-Elasticsearch-백업로그-검증-알림-DAG)]] — ES 기반 백업 검증
