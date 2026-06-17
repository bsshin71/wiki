# Airflow 파티션 add/remove 완료확인 알림 DAG

- **카테고리**: #airflow #usecase
- **태그**: #airflow #usecase #dag #SSHOperator #partition #notification #telegram
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-add partitionremove partition 완료 여부 확인 - BigDataTeam]]

## 1. 핵심 요약

원격 DB 서버의 **add/remove partition 작업 로그를 grep해 성공/실패를 판정**하고, PythonOperator로 메시지를 만들어 모니터링 서버를 통해 Telegram 알림을 보내는 DAG. 패턴: `SSHOperator(로그 grep) → PythonOperator(결과 판정·메시지) → SSHOperator(알림 발송)`.

---

## 2. DAG 구조

```
check_add_partition(SSH) >> check_if_success(Python) >> send_telegram(SSH)
```

```python
default_args = {"retries": 3, "retry_delay": pendulum.duration(minutes=10)}

check_add_partition = SSHOperator(
    task_id="check_add_partition", ssh_conn_id="sbosdb3",
    command="""
    cd /home/mysql/work;
    today=$(date +%Y%m%d);
    grep -A 999999 "^$today" addpart.log > /dev/null && echo "success" || echo "fail";
    """,
    cmd_timeout=10, do_xcom_push=True)

def check_result(**context):
    result = base64.b64decode(context["ti"].xcom_pull(task_ids="check_add_partition")).decode("utf-8").strip()
    msg = "✅ 파티션 추가 성공" if result=="success" else "⚠ 파티션 추가 실패"
    context["ti"].xcom_push(key="message", value=f"{msg} at {pendulum.now('UTC')}")

send_message = SSHOperator(
    task_id="send_telegram", ssh_conn_id="monitoring",
    command='cd /home/bos/temporary-notification; ./noti_personal.sh "{{ ti.xcom_pull(task_ids="check_if_success", key="message") }}";')
```

- `schedule_interval="0 23 * * *"` 매일 23시 점검.
- remove partition도 `removepart.log` 대상으로 동일 패턴.

## 3. 연관 개념

- [[2026-06-14-AF09_(Airflow-SSHOperator-리턴값-처리-xcom-retry)]] — grep+base64 xcom 패턴
- [[2026-06-14-AF12_(Airflow-DB백업-완료확인-멀티인스턴스-DAG)]]
- [[2026-06-14-AF10_(Airflow-Trigger-Rule)]]
