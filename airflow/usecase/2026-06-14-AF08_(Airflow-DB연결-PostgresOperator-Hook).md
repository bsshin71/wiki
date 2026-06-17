# Airflow DB 사용 (PostgresOperator · Hook)

- **카테고리**: #airflow #usecase
- **태그**: #airflow #usecase #dag #postgresql #operator #hook #SQLExecuteQueryOperator
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Database 사용 (PostgresOperator) - BigDataTeam]]

## 1. 핵심 요약

Airflow에서 DB 작업은 **Operator(SQL 파일 실행)** 와 **Hook(코드에서 직접 연결·쿼리)** 두 방식이 있다. 최신 버전은 DB 종류별 Operator(PostgresOperator 등) 대신 **`SQLExecuteQueryOperator` 하나로 통합**되었고, `conn_id`로 Connection을 지정한다. Connection은 Web UI Admin 또는 CLI로 등록한다.

---

## 2. Operator 방식

```python
# old: DB 종류별 개별 Operator
from airflow.providers.postgres.operators.postgres import PostgresOperator
write_to_postgres = PostgresOperator(
    task_id="write_to_postgres",
    postgres_conn_id="my_dw_postgres",
    sql="postgres_query.sql", dag=dag)

# latest: 통합 Operator (권장)
from airflow.providers.common.sql.operators.sql import SQLExecuteQueryOperator
write_to_postgres = SQLExecuteQueryOperator(
    task_id="write_to_postgres",
    conn_id="my_dw_postgres",
    sql="postgres_query.sql", dag=dag)
```

## 3. Hook 방식 (코드에서 직접 쿼리)

```python
from airflow.providers.postgres.hooks.postgres import PostgresHook
postgres_hook = PostgresHook(postgres_conn_id="postgres_con")

count_result = postgres_hook.get_first(f"""
    SELECT COUNT(*) FROM orders WHERE DATE(Timestamp) = '{yesterday}';""")
if count_result and count_result[0] > 0:
    postgres_hook.run("DELETE FROM orders WHERE DATE(Timestamp) = '{yesterday}';")
```

| 방식 | 메서드 | 용도 |
|------|--------|------|
| `get_first` | 단일 행 조회 | COUNT 등 |
| `get_records` | 다중 행 조회 | 데이터 추출 |
| `run` | DML 실행 | INSERT/DELETE |
| `insert_rows` | 행 일괄 삽입 | 적재 |

## 4. 연관 개념

- [[2026-06-14-AF14_(Airflow-MySQL-PostgreSQL-데이터전송-ETL)]] — Hook으로 DB간 전송
- [[2026-06-14-AF04_(Airflow-기본-Usecase-SSHOperator-Telegram)]]
