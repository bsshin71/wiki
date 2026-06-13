# Kafka 설치 (Apache / Confluent + Debezium CDC)

- **카테고리**: #kafka #install
- **태그**: #kafka #설치 #confluent #debezium #CDC #connect
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-kafka 설치 - BigDataTeam]]

## 1. 핵심 요약

Kafka는 **Apache Kafka**(최신 Debezium·간결)와 **Confluent Community**(ksql 등 유틸·간편 설치) 중 선택해 설치합니다.
설치 후 zookeeper→kafka(→schema-registry→connect) 순으로 기동하며, CDC 파이프라인은 Debezium MySQL source + Confluent JdbcSinkConnector 조합을 사용합니다.

---

## 2. 배포판 선택

| 기준 | 선택 |
|------|------|
| 최신 Debezium 사용 | Apache Kafka |
| 간편 설치 / ksql 등 유틸 | Confluent Community |

## 3. Apache Kafka 설치

```bash
# 다운로드·심볼릭 링크
ln -s pkg/kafka_2.13-3.6.1 kafka

# 환경변수(.bash_profile): JAVA_HOME 설정
# config/server.properties: advertised.listeners를 consumer/producer 접속 IP로 변경
# bin/kafka-server-start.sh: KAFKA_HEAP_OPTS 조정 (512M는 성능 저하 → 서버 스펙 맞게 상향)

nohup bin/kafka-server-start.sh config/server.properties > kafka.log &
```

테스트:
```bash
kafka-topics.sh --create --bootstrap-server 10.0.2.8:9092 --topic myuser
kafka-console-producer.sh --bootstrap-server 10.0.2.8:9092 --topic myuser
kafka-console-consumer.sh --bootstrap-server 10.0.2.8:9092 --topic myuser --from-beginning
```

## 4. Confluent Community 설치

```bash
curl -O https://packages.confluent.io/archive/7.5/confluent-community-7.5.3.tar.gz
tar xzvf confluent-community-7.5.3.tar.gz && ln -s confluent-7.5.3 confluent
# start 스크립트
bin/zookeeper-server-start -daemon etc/kafka/zookeeper.properties
bin/kafka-server-start -daemon etc/kafka/server.properties
```

## 5. Connector 설치 (Debezium CDC)

```bash
# JDK 11+ 필요 (최신 debezium)
# Confluent Hub client 설치
wget https://client.hub.confluent.io/confluent-hub-client-latest.tar.gz

# Debezium MySQL connector 설치
bin/confluent-hub install debezium/debezium-connector-mysql:2.2.1 \
  --component-dir /home/omegaman/confluent-hub/plugins \
  --worker-configs .../connect-distributed.properties
# → plugin.path에 plugins 경로 자동 추가

# Connect 기동
bin/connect-distributed etc/kafka/connect-distributed.properties
```

설치 확인:
```bash
curl localhost:8083/connector-plugins | jq
# "io.debezium.connector.mysql.MySqlConnector" (type source) 출력되면 정상
```

- **source**: `io.debezium.connector.mysql.MySqlConnector`
- **sink**: `io.confluent.connect.jdbc.JdbcSinkConnector` (자료 풍부, debezium JdbcSink 대비 권장)

## 6. CDC용 source DB 계정

```sql
CREATE USER `cdcuser`@`%` IDENTIFIED BY 'cdc1234';
GRANT SELECT, RELOAD, SHOW DATABASES, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO `cdcuser`@`%`;
FLUSH PRIVILEGES;
```

## 7. 연관 개념

- [[2026-06-14-K07_(Kafka-Broker-server.properties-설정)]]
- [[2026-06-14-K09_(Kafka-기동-순서-모듈)]]
- [[2026-06-14-K01_(Kafka-Connect-설치-confluent-hub)]]
- [[2026-06-14-K02_(Kafka-Connect-설정-Debezium-CDC)]]
