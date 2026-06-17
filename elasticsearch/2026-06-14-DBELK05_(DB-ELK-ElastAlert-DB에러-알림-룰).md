# DB-ELK ElastAlert — DB 에러 알림 룰 파일

- **카테고리**: #elasticsearch #monitoring
- **태그**: #elasticsearch #monitoring #elastalert #알림 #mysql-error #slack #webhook
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-elastalert db rule 파일 - BigDataTeam]]

## 1. 핵심 요약

ElastAlert가 ES 인덱스(`swz-myerr-*`)를 주기적으로 쿼리해 조건(1분 내 ERROR 1건 이상) 만족 시 **Slack·custom HTTP webhook**으로 알림을 발송한다. 룰 파일(YAML)에 필터 조건·알림 채널·메시지 템플릿을 정의한다.

---

## 2. devstg_myerr_error.yaml (에러 탐지 룰)

```yaml
name: "[dev/stg db] error detected in mysql-err.log"
type: any
index: swz-myerr-*

filter:
  - query:
      query_string:
        query: 'log_level:ERROR AND (env_type:dev OR env_type:stg)'

timeframe:
  minutes: 1         # 최근 1분 내 1건 이상이면 알림

alert:
  - "custom_alert.CustomPythonAlerter"   # custom HTTP webhook
  - "slack"

# custom webhook 설정
alarm_url: "http://10.11.6.147:7000/v1/alarmjson"
svcid: 8
alarmlevel: "warn"
category: "mysqlerr"

# 알림 메시지 템플릿
alert_subject: "🔥An ERROR detected in mysql-err.log of the host {1} service {0}"
alert_subject_args: [svc_name, host.name]
alert_text: |
  Env: {0} / Service: {1} / Host: {2}
  LogLevel: {3} / ErrNo: {4} / Module: {5}
  Errmsg: {6}
alert_text_args: [env_type, svc_name, host.name, log_level, errno, module, msg]

# Slack
slack_webhook_url: "https://hooks.slack.com/..."
slack_channel_override: "#warn"

realert:
  minutes: 1         # 1분 내 중복 알림 억제
```

## 3. 주요 룰 유형

| 룰 파일 | 조건 | 알림 채널 |
|---------|------|-----------|
| `devstg_myerr_error.yaml` | 1분 내 ERROR 1건 이상 | Slack #warn + webhook |
| `devstg_myerr_note.yaml` | 5분 내 NOTE 10건 이상 | Slack #info |
| (backup 체크) | 백업 성공 로그 없음 | webhook 에러 알림 |

## 4. 연관 개념

- [[2026-06-14-DBELK01_(DB-ELK-Docker-Compose-통합구성)]]
- [[2026-06-14-DBELK04_(DB-ELK-Logstash-파이프라인-구성)]]
- [[2026-06-14-AF13_(Airflow-Elasticsearch-백업로그-검증-알림-DAG)]] — ES 쿼리로 백업 결과 알림
