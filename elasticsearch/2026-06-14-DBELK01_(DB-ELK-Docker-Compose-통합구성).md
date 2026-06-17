# DB-ELK Docker Compose 통합 구성 (ES+Kibana+Logstash)

- **카테고리**: #elasticsearch #monitoring
- **태그**: #elasticsearch #monitoring #docker-compose #ELK #Kibana #Logstash #SSL
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-db-elk docker 구성 - BigDataTeam]]

## 1. 핵심 요약

DB 로그(MySQL error/slow log 등)를 **filebeat → logstash → Elasticsearch**로 수집·저장하는 ELK 스택을 Docker Compose로 구성한다. `setup` 서비스가 TLS 인증서를 자동 생성하고, `es01`(9200)·`kibana`(5601)·`logstash01`(5044/9600)이 SSL로 연결된다. `fleet-server` 포함 버전은 APM/Agent 모니터링 확장용.

---

## 2. 서비스 구성 (fleet-server 미포함 — 실운용)

| 서비스 | 이미지 | 포트 | 역할 |
|--------|--------|------|------|
| setup | elasticsearch:${STACK_VERSION} | — | 인증서 생성·kibana_system 비밀번호 설정 |
| es01 | elasticsearch:${STACK_VERSION} | 9200 | 단일 노드 ES (SSL, xpack.security) |
| kibana | kibana:${STACK_VERSION} | 5601 | 대시보드 (SSL) |
| logstash01 | logstash:${STACK_VERSION} | 5044(beats) / 9600 | filebeat 수신 → ES 적재 |

## 3. .env 주요 변수

```ini
STACK_VERSION=8.17.0
ELASTIC_PASSWORD=changeme
KIBANA_PASSWORD=changeme
ES_PORT=9200
KIBANA_PORT=5601
ES_MEM_LIMIT=4294967296   # 4G
KB_MEM_LIMIT=1073741824   # 1G
LICENSE=basic              # basic 또는 trial
```

## 4. 인증서 생성 (setup 서비스 자동)

```bash
# setup 서비스가 CA → es01/kibana/fleet-server 인증서 순서로 생성
# 결과: config/certs/ 아래 ca/, es01/, kibana/ 디렉터리
```

## 5. 연관 개념

- [[2026-06-14-DBELK02_(DB-ELK-Elasticsearch-인덱스-ILM-템플릿-설정)]]
- [[2026-06-14-DBELK03_(DB-ELK-Filebeat-DB로그-수집-설정)]]
- [[2026-06-14-DBELK04_(DB-ELK-Logstash-파이프라인-구성)]]
- [[2026-06-14-DBELK05_(DB-ELK-ElastAlert-DB에러-알림-룰)]]
