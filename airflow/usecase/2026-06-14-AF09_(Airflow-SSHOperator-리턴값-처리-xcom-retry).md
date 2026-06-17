# Airflow SSHOperator 리턴값 처리 (xcom · retry)

- **카테고리**: #airflow #usecase
- **태그**: #airflow #usecase #dag #SSHOperator #xcom #retry #base64
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-airflow sshoperator 리턴 테스트 - BigDataTeam]]

## 1. 핵심 요약

SSHOperator로 원격 스크립트 실행 시 **exit 0=성공 / exit 1=실패**로 판정되며, `do_xcom_push=True`로 stdout을 XCom에 push해 후행 PythonOperator에서 `xcom_pull`로 받는다. XCom 값은 **base64로 인코딩**되어 있어 디코딩이 필요하다. 실패 처리는 ① DAG `retries`로 재시도하거나 ② retry 없이 로그를 grep해 결과만 받는 방식이 있다.

---

## 2. 방법 1 — retry 사용

스크립트가 `exit 0/1`을 반환 → `retries`/`retry_delay`로 재시도.

```python
default_args = {"retries": 1, "retry_delay": timedelta(seconds=10)}

def get_ssh_output(**context):
    ssh_output = context["ti"].xcom_pull(task_ids="ssh_task")
    print(f"SSH 명령 결과: {ssh_output}")

ssh_task = SSHOperator(
    task_id="ssh_task", ssh_conn_id="detcdb1",
    command="cd /home/bos/for-airflow-test; ./test.sh;",
    cmd_timeout=5*60, do_xcom_push=True)      # exit 1 → 10초 후 1회 재시도
ssh_task >> get_output_task
```

## 3. 방법 2 — retry 없이 crontab 로그 grep

스크립트는 항상 성공하고, **오늘자 로그를 grep해 결과 텍스트를 XCom으로 회수** → base64 디코딩.

```python
import base64
def get_ssh_output(**context):
    ssh_output = context["ti"].xcom_pull(task_ids="grep_ssh_task")
    print(base64.b64decode(ssh_output).decode("utf-8"))

grep_ssh_task = SSHOperator(
    task_id="grep_ssh_task", ssh_conn_id="detcdb1",
    command='cd /home/bos/for-airflow-test; today=$(date +%F); grep -A 999999 "^$today" test.log;',
    cmd_timeout=5*60, do_xcom_push=True)
```

> ⚠️ XCom으로 받은 SSH 출력은 base64 → `base64.b64decode(...).decode("utf-8")` 필요.

## 4. 연관 개념

- [[2026-06-14-AF04_(Airflow-기본-Usecase-SSHOperator-Telegram)]]
- [[2026-06-14-AF11_(Airflow-파티션-완료확인-알림-DAG)]]
- [[2026-06-14-AF10_(Airflow-Trigger-Rule)]]
