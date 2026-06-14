# Elasticsearch ElastAlert 알림 도구

- **카테고리**: #elasticsearch #monitoring
- **태그**: #elasticsearch #elastalert #alert #docker #monitoring
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-ElastAlert - BigDataTeam]]

## 1. 핵심 요약

ElastAlert(Yelp 개발)은 **Elasticsearch 쿼리 기반으로 조건 충족 시 이메일·Slack·Webhook 등으로 알림**을 보내는 경량 알림 도구입니다.
룰은 YAML로 정의하며, Docker(`jertel/elastalert2`)로 구동하고 custom_alert.py로 통합 알람 서버에 전송합니다.

---

## 2. 특징

- **ES 통합**: ES 쿼리로 경고 조건 정의.
- **다양한 알림**: 이메일/Slack/Telegram/Webhook.
- **룰 타입**: frequency(빈도), match absence(부재), spike(급증) 등.
- **YAML 룰 설정**.

> 예: "5분간 로그인 실패 10회 이상 → Slack", "error 로그 하루 100건 이상 → 이메일".

## 3. 설치 (Docker)

### 디렉토리 구조
```
elastalert/
├── bin/        # custom_alert.py
├── config/     # config.yaml, supervisord.conf
├── logs/
└── rules/      # db, proxysql, system, web 별 룰
```

### Dockerfile (jertel/elastalert2)
```dockerfile
FROM jertel/elastalert2:latest
USER root
COPY ./bin/custom_alert.py /opt/elastalert/
RUN apt-get update && apt-get install -y supervisor && rm -rf /var/lib/apt/lists/*
RUN python -m pip install requests jinja2
# myrun.sh: elastalert-create-index 후 supervisord 실행
WORKDIR /opt/elastalert
ENTRYPOINT ["/opt/elastalert/myrun.sh"]
```
- `custom_alert.py`: python 함수로 통합 알람 서버에 alert 전송.
- `myrun.sh`: supervisord로 elastalert 프로세스 실행 + config 전달.

### docker-compose.yml
```yaml
services:
  elastalert:
    build: .
    container_name: elastalert
    volumes:
      - ./config:/opt/config
      - ./rules:/opt/rules
      - ./logs:/opt/logs
      - /etc/logstash/conf.d/http_ca.crt:/etc/logstash/conf.d/http_ca.crt:ro
    network_mode: host    # localhost:9200 접근 위해 host 모드
    restart: unless-stopped
```

## 4. 연관 개념

- [[2026-06-14-K18_(Kafka-모니터링-Prometheus-Alertmanager)]]
- [[2026-06-14-ES04_(Elasticsearch-API-Key-연결)]]
