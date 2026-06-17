# DB-ELK Logstash 파이프라인 구성

- **카테고리**: #elasticsearch #monitoring
- **태그**: #elasticsearch #monitoring #logstash #pipeline #beats #DB로그
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-db-elk logstash 구성 - BigDataTeam]]

## 1. 핵심 요약

Logstash가 **Filebeat(5044)에서 로그를 수신**하고 파이프라인(main→db/system/app)으로 분류해 Elasticsearch에 적재한다. 멀티 파이프라인 구조: `main.conf`가 beats 입력을 받아 `msg_type` 필드로 sub 파이프라인에 분기한다.

---

## 2. 디렉터리 구조

```
logstash/
├── config/   logstash.yml, pipelines.yml, log4j2.properties
├── pipelines/
│   ├── main/   main.conf (beats 수신 + 분기)
│   └── db/     db.conf (MySQL 로그 파싱 → ES 적재)
└── data/     queue, dead_letter_queue 등
```

## 3. main.conf (beats 수신 + 파이프라인 분기)

```ruby
input {
  beats {
    port => 5044
    client_inactivity_timeout => 300
  }
}
filter {
  # msg_type 필드로 sub 파이프라인 분기 처리
}
output {
  if [msg_type] == "myerr" {
    pipeline { send_to => "db-pipeline" }
  } else if [msg_type] == "myslow" {
    pipeline { send_to => "db-pipeline" }
  }
}
```

## 4. db.conf (MySQL 로그 파싱 → ES 적재)

```ruby
input { pipeline { address => "db-pipeline" } }
filter {
  # MySQL error log: YYYY-MM-DDTHH:MM:SS.ffffffZ N [Level] [Code] [Component] msg 파싱
  if [msg_type] == "myerr" {
    grok { match => { "message" => "..." } }
  }
}
output {
  elasticsearch {
    hosts => "${ELASTIC_HOSTS}"
    user => "${ELASTIC_USER}"
    password => "${ELASTIC_PASSWORD}"
    index => "swz-myerr-%{+YYYY.MM}"
    ssl_certificate_authorities => "certs/ca/ca.crt"
  }
}
```

## 5. 연관 개념

- [[2026-06-14-DBELK03_(DB-ELK-Filebeat-DB로그-수집-설정)]]
- [[2026-06-14-DBELK02_(DB-ELK-Elasticsearch-인덱스-ILM-템플릿-설정)]]
- [[2026-06-14-DBELK05_(DB-ELK-ElastAlert-DB에러-알림-룰)]]
