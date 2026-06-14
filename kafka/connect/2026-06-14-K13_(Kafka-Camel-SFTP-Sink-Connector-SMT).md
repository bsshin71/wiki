# Kafka Camel SFTP Sink Connector (+ 커스텀 SMT)

- **카테고리**: #kafka #connect
- **태그**: #kafka #connect #sink #sftp #camel #SMT
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-camel sftp sink connector - BigDataTeam]]

## 1. 핵심 요약

Apache Camel `CamelSftpSinkConnector`로 Kafka 토픽을 SFTP 서버에 일자별 디렉토리로 적재합니다.
파일 개행 추가·태스크별 파일명 지정을 위해 **커스텀 SMT 2종(AppendNewline, SetFileNameFromTask)** 을 직접 컴파일해 사용하며, 커넥터는 add/update/delete/status 셸 스크립트로 관리합니다.

---

## 2. 설치

```bash
# camel-sftp-kafka-connector 다운로드
https://repo1.maven.org/maven2/org/apache/camel/kafkaconnector/camel-sftp-kafka-connector/0.11.5/camel-sftp-kafka-connector-0.11.5-package.tar.gz
```

## 3. 커스텀 SMT 컴파일

```bash
# CLASSPATH에 connect-api jar 포함 후 javac → jar
CLASSPATH="$CONFLUENT/share/java/kafka/*:.../connect-api-7.9.2-ccs.jar"
javac -cp "$CLASSPATH" com/softwiz/kafka/transforms/AppendNewline.java
jar cf softwiz-append-newline-smt.jar -C . com/softwiz/kafka/transforms/*.class
```

| SMT | 역할 |
|-----|------|
| `AppendNewline` | record value(String 전제)에 개행문자 추가, value 스키마 STRING 덮어쓰기 옵션 |
| `SetFileNameFromTask` | MDC `connector.context`의 `task-N`으로 `yyyyMMdd_HH_t<task>.log` 파일명 생성 |

## 4. Connector 설정 (핵심)

```json
"connector.class": "org.apache.camel.kafkaconnector.sftp.CamelSftpSinkConnector",
"topics": "bos.useractlog",
"camel.sink.path.host": "sbosdw1", "camel.sink.path.port": "2121",
"camel.sink.path.directoryName": "/data/${date:now:yyyy}/${date:now:MM}/${date:now:dd}",
"camel.sink.endpoint.fileExist": "Append",
"camel.sink.endpoint.autoCreate": "true",
"consumer.override.max.poll.records": "2000",
"value.converter": "org.apache.kafka.connect.storage.StringConverter",
"transforms": "SetFileName,AppendNewline",
"transforms.AppendNewline.type": "com.softwiz.kafka.transforms.AppendNewline",
"transforms.AppendNewline.newline": "\n",
"transforms.SetFileName.type": "com.softwiz.kafka.transforms.SetFileNameFromTask"
```
> `camel.sink.path.directoryName`의 `${date:now:...}`로 일자별 디렉토리 자동 생성, `fileExist=Append`로 같은 파일에 누적.

## 5. Connector 관리 스크립트

```bash
# sftp_sink3.sh {add|update|delete|status}
# add: 존재 시 DELETE 후 POST / update: PUT .../config / status: GET .../status | jq
curl -s -X POST -H "Content-Type: application/json" --data @config.json "$CONNECT_URL"
curl -s -X PUT  --data @config.json "$CONNECT_URL/${CONN_NAME}/config"
curl -s -X DELETE "$CONNECT_URL/${CONN_NAME}"
```

## 6. 연관 개념

- [[2026-06-14-K12_(Kafka-S3-Sink-Connector)]]
- [[2026-06-14-K11_(Kafka-Elasticsearch-Sink-Connector)]]
- [[2026-06-14-K02_(Kafka-Connect-설정-Debezium-CDC)]]
