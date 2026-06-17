# Airflow Elasticsearch 백업로그 검증 알림 DAG

- **카테고리**: #airflow #usecase
- **태그**: #airflow #usecase #dag #elasticsearch #backup #monitoring #webhook
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-elasticsearch 로그 검색결과를 통한 backup 결과 전송 - BigDataTeam]]

## 1. 핵심 요약

각 DB 서버 백업 로그가 **filebeat→logstash→Elasticsearch**로 적재되면, Airflow가 ES를 쿼리해 **어제 일자 "Backup process completed: Success" 로그가 있는 호스트**를 조회하고, 대상 인스턴스 집합과 비교해 성공/실패를 분류 후 통합 알람 서버(HTTP webhook)로 전송한다. ES 연결은 Airflow Connection(`db-elk`)으로 관리하며, Docker 실행 시 **ES 인증서를 컨테이너에 mount**해야 한다.

---

## 2. 데이터 흐름

```
각 서버 backup script(crontab) → 백업로그
  → filebeat(변경 감지) → logstash → Elasticsearch(swz-backuplog-*)
  → Airflow가 ES 쿼리 → 성공 host 추출 → 대상 비교 → webhook 알림
```

## 3. ES 쿼리 (어제 성공 로그 호스트)

```python
search_body = {"query": {"bool": {"must": [
    {"range": {"@timestamp": {"gte": yesterday_start.isoformat(), "lte": yesterday_end.isoformat()}}},
    {"term": {"module": "MAIN"}},
    {"match_phrase": {"msg": "Backup process completed: Success"}}
]}}, "fields": ["host.name"], "_source": False, "size": 1000}
```

- 성공 호스트 = `target_instances ∩ queried_hosts`, 실패 = `target_instances - queried_hosts`.

## 4. Docker Compose — ES 인증서 mount

```yaml
volumes:
  - ${AIRFLOW_PROJ_DIR:-.}/dags:/opt/airflow/dags
  - ${AIRFLOW_PROJ_DIR:-.}/certs:/opt/airflow/dags/certs   # ← ES 인증서 추가
```
- `AIRFLOW__CORE__TEST_CONNECTION: 'Enabled'` (UI 연결 테스트 허용)

## 5. 알림 (HTTP webhook)

```python
url = "http://10.11.6.147:7000/v1/alarmjson"
payload = {"svcid":1, "service":"airflow", "subject":"PRD DB Backup Result", "category":"monitoring", ...}
requests.post(url, json=payload)   # 성공/실패 리스트별 발송
```

## 6. 연관 개념

- [[2026-06-14-AF12_(Airflow-DB백업-완료확인-멀티인스턴스-DAG)]] — SSH 로그 직접 점검 방식
- [[2026-06-14-AF08_(Airflow-DB연결-PostgresOperator-Hook)]]
