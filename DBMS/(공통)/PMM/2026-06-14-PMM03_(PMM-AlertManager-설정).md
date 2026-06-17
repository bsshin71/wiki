# PMM AlertManager 설정

- **카테고리**: #DBMS #monitoring
- **태그**: #PMM #alertmanager #prometheus #monitoring #slack #webhook
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-PMM alertmanager - BigDataTeam]]

## 1. 핵심 요약

PMM(Percona Monitoring and Management)에 연동된 **Prometheus AlertManager**로 DB 장애(MysqlDown 등) 알림을 Slack·webhook으로 발송한다. `alertmanager.yml`의 global·route·receivers·inhibit_rules·templates 5개 섹션으로 구성하며, PMM이 HTTP POST JSON으로 alert를 전달한다.

---

## 2. alertmanager.yml 구조

```yaml
global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/services/...'

route:                    # alert 라우팅 트리
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 1h
  receiver: 'slack-telegrambot'

receivers:
- name: 'slack-telegrambot'
  webhook_configs:
  - url: 'http://127.0.0.1:8080/'   # 자체 webhook receiver
    send_resolved: true
  slack_configs:
  - channel: '#alert'
    send_resolved: true

inhibit_rules:            # 상위 severity 발생 시 하위 억제
  - source_match: {severity: 'critical'}
    target_match: {severity: 'warning'}
    equal: ['alertname', 'dev', 'instance']
```

## 3. PMM Alert JSON 구조 (MysqlDown 예시)

```json
{
  "alerts": [{
    "labels": {
      "alertname": "MysqlDown",
      "service_name": "test-db",
      "service_type": "mysql",
      "severity": "critical",
      "node_name": "DBMS-MONITOR01"
    },
    "annotations": {
      "summary": "MySQL down (instance ...)",
      "description": "MySQL instance is down ... VALUE = 0"
    },
    "status": "firing",
    "startsAt": "2024-...",
    "endsAt": "0001-01-01T00:00:00Z"
  }]
}
```

## 4. 연관 개념

- [[2026-06-14-PMM04_(PMM-모니터링-서버-구축-Docker)]]
- [[2026-06-14-MON02_(DB-장애-알림-시스템-단계별-설계)]]
