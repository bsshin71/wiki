# Kafka Connect 설치 (confluent-hub)

- **카테고리**: #kafka #connect
- **태그**: #kafka #connect #connector #confluent-hub #설치
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-connect 설치 - BigDataTeam]]

## 1. 핵심 요약

Kafka Connect 커넥터는 **`confluent-hub install`** 명령으로 Confluent Hub에서 받아 설치하며, 설치 시 worker 설정 파일들의 `plugin.path`에 컴포넌트 디렉토리가 자동 추가됩니다.
`confluent-hub` CLI는 Confluent 패키지에 기본 포함되어 있습니다.

---

## 2. confluent-hub 설치

- Confluent Platform 패키지에 기본 포함.
- 수동 설치도 가능: https://docs.confluent.io/current/connect/managing/confluent-hub/client.html

## 3. 커넥터 설치 (예: Debezium MySQL CDC)

```bash
confluent-hub install debezium/debezium-connector-mysql:1.2.2
```

설치 대화 과정:
1. 설치 위치 선택 (`$CONFLUENT_HOME` 기반, 예 `/home/bos/confluent`)
2. 컴포넌트 디렉토리 확인 → `share/confluent-hub-components`
3. 라이선스 동의
4. **Worker 설정 일괄 업데이트(y)** → 아래 4개 파일의 `plugin.path`에 설치 경로 자동 추가:
   - `etc/kafka/connect-distributed.properties`
   - `etc/kafka/connect-standalone.properties`
   - `etc/schema-registry/connect-avro-distributed.properties`
   - `etc/schema-registry/connect-avro-standalone.properties`

## 4. 참고

- 커넥터 검색: https://www.confluent.io/hub/

## 5. 연관 개념

- [[2026-06-14-K02_(Kafka-Connect-설정-Debezium-CDC)]]
- [[2026-06-14-K03_(Kafka-Connect-Property-분산모드-설정)]]
