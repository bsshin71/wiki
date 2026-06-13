# Kafka kcat (kafkacat) 도구

- **카테고리**: #kafka #utility
- **태그**: #kafka #kcat #kafkacat #producer #consumer #utility
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-kcat(kafkacat) - BigDataTeam]], [[2026-06-13-kafkacat - BigDataTeam]]

## 1. 핵심 요약

kcat(구 kafkacat)은 producer·consumer·metadata 조회를 제공하는 강력한 CLI 도구로, `kafka-console-*`보다 풍부한 기능(포맷 문자열, 오프셋/타임스탬프 쿼리, mock cluster)을 지원합니다.
`-P`(produce)/`-C`(consume)/`-L`(metadata)/`-Q`(query) 모드와 `-f` 포맷 문자열이 핵심입니다.

---

## 2. 설치

```bash
# 의존성 + 빌드 (방법 A)
yum install -y gcc-c++ git librdkafka-devel
git clone https://github.com/edenhill/kafkacat && cd kafkacat
./configure && make && make install

# 방법 B (librdkafka yum + prefix 설치)
sudo yum install librdkafka*
git clone https://github.com/edenhill/kafkacat kafkacat-src && cd kafkacat-src
./configure --prefix=$HOME/packages/kafkacat && make && make install

kcat -V   # 버전 확인
```

## 3. 기본 사용 (모드)

```bash
# Producer (-P)
kcat -P -b localhost:9092 -t test

# Consumer (-C, 기본 auto-select)
kcat -b localhost:9092 -t test

# High-level consumer group (-G)
kcat -b <broker> -G <group-id> topic1 topic2

# Metadata list (-L)
kcat -L -b localhost:9092

# Query offset by timestamp (-Q)
kcat -Q -b broker -t <topic>:<partition>:<timestamp>
```

## 4. connect-offsets 토픽 조회 (포맷 문자열)

```bash
# 전체
kcat -b localhost:9092 -C -t connect-offsets -f 'Partition(%p) %k %s\n'
# 특정 파티션/오프셋
kcat -b localhost:9092 -C -t connect-offsets -f 'Partition(%p) %k %s\n' -o 10537 -p 3
# 마지막
kcat -b localhost:9092 -C -t connect-offsets -f 'Partition(%p) %k %s\n' -o end
```

## 5. 주요 옵션

| 옵션 | 의미 |
|------|------|
| `-o` | offset: `beginning`/`end`/`stored`/절대/상대(`-N`)/`s@ts`/`e@ts` |
| `-f` | 출력 포맷(`%t %p %o %k %s %T %h`) |
| `-K`/`-D` | key/message 구분자 |
| `-z snappy\|gzip\|lz4` | producer 압축 |
| `-X prop=val` | librdkafka 설정 (`-X list`로 목록) |
| `-F <config-file>` | 설정 파일(`$HOME/.config/kcat.conf`) |
| `-e` | 마지막 메시지 수신 시 정상 종료 |

> 포맷 예: `-f 'Topic %t [%p] at offset %o: key %k: %s\n'`

## 6. 연관 개념

- [[2026-06-14-K10_(Kafka-Topic-Producer-Consumer-명령어)]]
- [[2026-06-14-K17_(Kafka-UI-tool-provectus)]]
- [[2026-06-14-K05_(Kafka-Connect-Sink-Connector-시작-offset-제어)]]
