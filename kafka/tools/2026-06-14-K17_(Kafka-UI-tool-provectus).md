# Kafka UI tool (provectus)

- **카테고리**: #kafka #utility
- **태그**: #kafka #UI #provectus #docker #모니터링
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-kafka-ui tool - BigDataTeam]]

## 1. 핵심 요약

provectus `kafka-ui`를 **docker compose**로 띄워 웹 UI(8989)에서 토픽·컨슈머·메시지를 조회합니다.
docker 컨테이너이므로 `BOOTSTRAPSERVERS`는 **localhost가 아닌 실제 Kafka 접속 IP**로 지정해야 합니다.

---

## 2. docker compose 설정

```yaml
version: '2'
services:
  kafka-ui:
    image: provectuslabs/kafka-ui
    container_name: kafka-ui
    ports:
      - "8989:8080"
    restart: always
    environment:
      - KAFKA_CLUSTERS_0_NAME=local
      - KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS=10.0.2.8:9092   # ⚠️ docker → localhost:9092 불가, 실제 IP
```

## 3. 시작 / 종료

```bash
# 시작
docker compose -f docker-compose-kafka-ui.yaml up -d
# 종료
docker compose -f docker-compose-kafka-ui.yaml down
# 웹 접속: http://<host>:8989
```

## 4. 에러 대처

```
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```
→ docker 데몬 미기동. 해결:
```bash
systemctl start docker
```

## 5. 연관 개념

- [[2026-06-14-K16_(Kafka-kcat-kafkacat-도구)]]
- [[2026-06-14-K18_(Kafka-모니터링-Prometheus-Alertmanager)]]
