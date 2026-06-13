# Kafka 모니터링 (Prometheus + Alertmanager)

- **카테고리**: #kafka #monitoring
- **태그**: #kafka #monitoring #prometheus #alertmanager #jmx_exporter
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13- alertmanager 를 이용한 alert - BigDataTeam]]

## 1. 핵심 요약

Kafka 모니터링은 **JMX exporter + node exporter → Prometheus(scrape) → Alertmanager(알림)** 구조로 구성합니다.
Prometheus `rules.yml`에 BrokerState·ActiveControllerCount·UncleanLeaderElection 등 알림 규칙을 정의하고, Alertmanager가 임계 위반 시 알림을 발송합니다.

---

## 2. 구조

```
[Kafka broker] --(jmx_exporter, node_exporter)--> [Prometheus] --(alert)--> [Alertmanager] --> 알림
```

## 3. Alertmanager 설치

```bash
wget https://github.com/prometheus/alertmanager/releases/download/v0.22.2/alertmanager-0.22.2.linux-amd64.tar.gz
ln -s alertmanager-0.22.2.linux-amd64 alertmanager
mkdir /etc/alertmanager && nano /etc/alertmanager/alertmanager.yml
# systemd unit: /etc/systemd/system/alertmanager.service (User=alertmanager)
systemctl daemon-reload && systemctl restart alertmanager   # 9093 포트
```

## 4. Prometheus 설정 (prometheus.yml)

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
alerting:
  alertmanagers:
  - static_configs:
    - targets: [ localhost:9093 ]     # alertmanager 주소
rule_files:
  - rules.yml                          # alert rule 파일
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
    - targets: ['localhost:9090']
  - job_name: 'broker'
    file_sd_configs:
    - files: [ 'targets.json' ]        # jmx/node exporter 대상
```

## 5. Alert Rule 예시 (rules.yml)

```yaml
groups:
- name: KafkaStatus
  rules:
  - alert: Kafka-InstanceDown
    expr: kafka_server_KafkaServer_Value{instance=~".*11001",name="BrokerState"}!=3
    for: 1m
    labels: { severity: 'critical' }
    annotations: { title: 'Instance {{ $labels.instance }} down' }

  - alert: Kafka-Down
    expr: up{job="jmx"}==0
    for: 1m
    labels: { severity: 'critical' }

  - alert: Kafka-ActiveControllerCount
    expr: sum(kafka_controller_KafkaController_Value{job="jmx",name=~"ActiveControllerCount"}) < 1
    for: 1m

  - alert: Kafka-UncleanLeaderElectionRate
    expr: sum(kafka_controller_ControllerStats_Count{job="jmx",name="UncleanLeaderElectionsPerSec"}) > 0
    for: 1m
```

> 설정된 alert는 `http://<prometheus>:9090/alerts`에서 확인. BrokerState=3(Running)이 정상.

## 6. 주요 모니터링 지표

| 지표 | 의미 |
|------|------|
| BrokerState | 브로커 상태(3=Running) |
| ActiveControllerCount | 활성 컨트롤러 수(<1이면 이상) |
| UncleanLeaderElectionsPerSec | unclean leader election(데이터 유실 위험) |
| OfflinePartitionCount | 오프라인 파티션 수 |

## 7. 연관 개념

- [[2026-06-14-K15_(Kafka-성능-튜닝-커널-디스크-GC)]]
- [[2026-06-14-K17_(Kafka-UI-tool-provectus)]]
