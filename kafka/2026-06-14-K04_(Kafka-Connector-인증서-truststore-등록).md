# Kafka Connector 인증서 truststore 등록

- **카테고리**: #kafka #connect
- **태그**: #kafka #connect #SSL #truststore #keytool #인증서
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-connector 에 사용하는 인증서 등록 - BigDataTeam]]

## 1. 핵심 요약

Kafka Connect는 **JVM truststore(`cacerts`)** 로 서버 인증서를 검증하므로, HTTPS 대상(예: Elasticsearch) 연동 시 `keytool -import`로 CA 인증서를 등록하고 **Connect를 재시작**해야 합니다.
JVM 전역 cacerts 대신 **별도 truststore.jks**를 만들어 JVM 옵션으로 지정할 수도 있습니다.

---

## 2. 방법 A — JVM 전역 truststore(cacerts)에 등록

```bash
# 인증서 위치: $JAVA_HOME/lib/security/cacerts (기본 비밀번호 changeit)
sudo keytool -import -trustcacerts -alias elasticsearch-ca \
    -file /home/confluent/cert/http_ca.crt \
    -keystore $JAVA_HOME/lib/security/cacerts \
    -storepass changeit

# 확인
keytool -list -keystore $JAVA_HOME/lib/security/cacerts -storepass changeit | grep elasticsearch-ca
```
→ **Connect 재시작 필요.**

| 옵션 | 의미 |
|------|------|
| `-alias` | 인증서 식별 이름 |
| `-file` | CA 인증서 파일 |
| `-keystore` | truststore 경로 |
| `-storepass` | truststore 비밀번호(기본 changeit) |

## 3. 방법 B — 별도 truststore.jks + JVM 옵션

```bash
keytool -import -trustcacerts -alias elasticsearch-ca \
    -file /home/confluent/cert/http_ca.crt \
    -keystore /home/confluent/cert/es-truststore.jks -storepass mypassword
```
```properties
# kafka connect JVM 옵션 (KAFKA_OPTS 또는 systemd)
-Djavax.net.ssl.trustStore=/home/confluent/cert/es-truststore.jks
-Djavax.net.ssl.trustStorePassword=mypassword
```

## 4. 삭제 / 갱신

```bash
keytool -delete -alias elasticsearch-ca \
  -keystore <cacerts경로> -storepass changeit
```
> JDK별 cacerts 경로 주의 (예: `/usr/lib/jvm/java-11-openjdk-.../lib/security/cacerts`).

## 5. 연관 개념

- [[2026-06-14-K03_(Kafka-Connect-Property-분산모드-설정)]]
- [[2026-06-14-K02_(Kafka-Connect-설정-Debezium-CDC)]]
