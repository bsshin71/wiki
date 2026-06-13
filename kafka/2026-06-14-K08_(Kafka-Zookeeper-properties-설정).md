# Kafka Zookeeper properties 설정

- **카테고리**: #kafka #config
- **태그**: #kafka #config #zookeeper #ensemble
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-zookeeper property - BigDataTeam]]

## 1. 핵심 요약

Zookeeper `zookeeper.properties`는 **tickTime·initLimit·syncLimit**(타이밍), **dataDir**(스냅샷 저장), **clientPort(2181)**, **server.N**(앙상블 멤버)로 구성됩니다.
`dataDir`은 `/tmp`를 쓰지 말고 영구 경로로 지정하고, 운영에서는 autopurge로 스냅샷을 정리합니다.

---

## 2. 주요 설정

```properties
tickTime=2000            # 기본 타이밍 단위(ms)
initLimit=10             # 초기 동기화 허용 tick 수
syncLimit=5              # 요청-응답 허용 tick 수

dataDir=/home/bos/confluent/data/zookeeper   # ⚠️ /tmp 사용 금지(영구 경로)
clientPort=2181          # 클라이언트 접속 포트

#maxClientCnxns=60       # 클라이언트 연결 수 제한
#autopurge.snapRetainCount=3   # 보존 스냅샷 수
#autopurge.purgeInterval=1     # 정리 주기(시간), 0=비활성

server.1=kafka:2888:3888 # 앙상블 멤버 (2888=peer, 3888=leader election)
```

> 멀티 노드 앙상블 시 `server.1`, `server.2`, `server.3`을 각 노드에 추가하고 각 `dataDir/myid`에 번호 지정.

## 3. 연관 개념

- [[2026-06-14-K07_(Kafka-Broker-server.properties-설정)]]
- [[2026-06-14-K09_(Kafka-기동-순서-모듈)]]
