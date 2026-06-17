# Airflow 기본 Usecase (SSHOperator + TelegramOperator)

- **카테고리**: #airflow #usecase
- **태그**: #airflow #usecase #dag #SSHOperator #TelegramOperator #trigger_rule
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-airflow 기본 usecase - BigDataTeam]]

## 1. 핵심 요약

**원격 서버의 스크립트를 SSHOperator로 실행하고, 성공/실패에 따라 TelegramOperator로 알림**을 보내는 DAG 패턴. `trigger_rule`(`all_success`/`one_failed`)로 후행 태스크의 실행 조건을 제어하고, `apache-airflow-providers-ssh`·`-telegram` 설치 + Admin > Connections 등록이 전제다.

---

## 2. 사전 준비

- `apache-airflow-providers-ssh`, `apache-airflow-providers-telegram` 설치
- **Admin > Connections** 에서 SSH connection 생성
- Telegram connection: Host=channel id, Password=bot token

## 3. DAG 예시 (MySQL 원격 백업 + 알림)

```python
from airflow import DAG
from airflow.providers.ssh.operators.ssh import SSHOperator
from airflow.providers.telegram.operators.telegram import TelegramOperator
from datetime import datetime, timedelta

with DAG("mysql_backup_noti", schedule_interval="@daily",
         start_date=datetime(2024,11,14), catchup=False) as dag:

    backup_task = SSHOperator(
        task_id="mysql_remote_backup",
        ssh_conn_id="my-instance",
        command="""
        cd /home/temp/mysql-backup;
        ./full_backup.sh;
        echo "Script execution complete"
        """,                       # 여러 줄로 작성(한 줄이면 jinja template 에러)
        cmd_timeout=3600,
    )
    send_success = TelegramOperator(task_id="send_success_telegram",
        telegram_conn_id="my-success-telegram",
        text="백업 완료", trigger_rule="all_success")
    send_failure = TelegramOperator(task_id="send_failure_telegram",
        telegram_conn_id="my-fail-telegram",
        text="백업 오류", trigger_rule="one_failed")

    backup_task >> [send_success, send_failure]
```

> ⚠️ SSHOperator `command`를 한 줄로 쓰면 jinja template 에러 → **여러 줄**로 작성.

## 4. DAG 등록

docker compose에서 `dags/` 디렉토리가 마운트돼 있으므로, 해당 디렉토리에 `.py` 파일을 두면 자동 인식된다.

## 5. 연관 개념

- [[2026-06-14-AF02_(Airflow-Docker-Compose-설치)]]
- [[2026-06-14-AF03_(Airflow-개발환경-VSCode-DevContainer)]]
