# DB-ELK Elasticsearch 인덱스 ILM·템플릿 설정

- **카테고리**: #elasticsearch #monitoring
- **태그**: #elasticsearch #monitoring #ILM #index-template #DB로그 #lifecycle
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-db-elk elasticsearch 구성 - BigDataTeam]]

## 1. 핵심 요약

DB 로그용 Elasticsearch 구성 절차: ① **ILM 정책**(보관 기간 지정) → ② **인덱스 템플릿**(shard/replica·ILM 연결·mapping). 각 로그 유형(mysql-err·mysql-slow·proxysql·backup 등)별로 정책과 템플릿을 분리 생성한다.

---

## 2. ILM Policy 생성 (보관 기간)

```json
// mysql-err.log: 10일 보관
PUT /_ilm/policy/swz-myerrlog-policy
{"policy":{"phases":{"delete":{"min_age":"10d","actions":{"delete":{}}}}}}

// mysql-slow.log: 10일 보관
PUT /_ilm/policy/swz-myslowlog-policy
{"policy":{"phases":{"delete":{"min_age":"10d","actions":{"delete":{}}}}}}

// proxysql log: 30일 보관
PUT /_ilm/policy/swz-proxylog-policy
{"policy":{"phases":{"delete":{"min_age":"30d","actions":{"delete":{}}}}}}
```

## 3. 인덱스 템플릿 (mysql-err 예시)

```json
PUT /_index_template/swz-myerrlog-template
{
  "index_patterns": ["swz-myerr-*"],
  "template": {
    "settings": {
      "index": {"number_of_shards":"4","number_of_replicas":"1",
                "refresh_interval":"5s","lifecycle":{"name":"swz-myerrlog-policy"}}
    },
    "mappings": {
      "properties": {
        "@timestamp": {"type":"date"},
        "log_level": {"type":"keyword"}, "errno": {"type":"keyword"},
        "module": {"type":"keyword"}, "msg": {"type":"text"},
        "host.name": {"type":"keyword"}, "env_type": {"type":"keyword"}
      },
      "dynamic_templates": [
        {"message": {"match_mapping_type":"string","match":"msg*","mapping":{"type":"text"}}},
        {"default_strings": {"match_mapping_type":"string","mapping":{"type":"keyword"}}}
      ]
    }
  }
}
```

## 4. 연관 개념

- [[2026-06-14-DBELK01_(DB-ELK-Docker-Compose-통합구성)]]
- [[2026-06-14-DBELK03_(DB-ELK-Filebeat-DB로그-수집-설정)]]
