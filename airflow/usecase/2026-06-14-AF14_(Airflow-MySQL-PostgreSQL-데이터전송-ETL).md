# Airflow MySQL→PostgreSQL 데이터 전송 (ETL)

- **카테고리**: #airflow #usecase
- **태그**: #airflow #usecase #dag #etl #MySqlHook #PostgresHook #telegram
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-mysql - postgresql 어제 일자 데이터 전송 - BigDataTeam]]

## 1. 핵심 요약

**MySqlHook으로 어제 일자 데이터를 조회하고 PostgresHook으로 적재**하는 일별 ETL DAG. 멱등성을 위해 적재 전 대상 PostgreSQL의 동일 일자 데이터를 DELETE 후 INSERT하며, 처리 시간·서버명을 `task_log`에 기록하고 성공/실패를 Telegram으로 알린다(`trigger_rule="one_failed"`로 실패 분기).

---

## 2. 핵심 로직

```python
from airflow.providers.mysql.hooks.mysql import MySqlHook
from airflow.providers.postgres.hooks.postgres import PostgresHook

def transfer_data(**kwargs):
    mysql_hook = MySqlHook(mysql_conn_id="mysql_con")
    postgres_hook = PostgresHook(postgres_conn_id="postgres_con")
    yesterday = pendulum.yesterday().strftime("%Y-%m-%d")

    # 1) 멱등성: 대상 동일 일자 데이터 선삭제
    if postgres_hook.get_first(f"SELECT COUNT(*) FROM orders WHERE DATE(Timestamp)='{yesterday}';")[0] > 0:
        postgres_hook.run("DELETE FROM orders WHERE DATE(Timestamp)='{yesterday}';")

    # 2) MySQL 조회 → PostgreSQL 적재
    records = mysql_hook.get_records(f"SELECT * FROM orders WHERE DATE(Timestamp)='{yesterday}';")
    if records:
        postgres_hook.insert_rows("orders", records)

    # 3) 처리 로그 기록 (소요시간·서버)
    postgres_hook.run(log_query, parameters=(...))
```

## 3. 알림 분기

```python
transfer_task >> [telegram_success_task, telegram_failure_task]
# 실패 task: trigger_rule="one_failed" → 성공 chat_id / 실패 chat_id 분리 발송
```

> `socket.gethostname()`은 docker 컨테이너명을 반환 → 호스트 서버명을 쓰려면 docker 실행 시 변수로 전달.

## 4. 연관 개념

- [[2026-06-14-AF08_(Airflow-DB연결-PostgresOperator-Hook)]] — Hook 메서드 상세
- [[2026-06-14-AF10_(Airflow-Trigger-Rule)]] — one_failed 분기
- [[2026-06-14-AF12_(Airflow-DB백업-완료확인-멀티인스턴스-DAG)]]
