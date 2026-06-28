# Elasticsearch API Key 연결

- **카테고리**: #elasticsearch #admin
- **태그**: #elasticsearch #apikey #security #curl #bulk
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-api key 를 통한 연결 - BigDataTeam]]

## 1. 핵심 요약

`POST /_security/api_key`로 역할(role_descriptors)을 지정한 API Key를 발급하고, 응답의 **`encoded`** 값을 클라이언트가 `Authorization: ApiKey <encoded>` 헤더로 사용합니다.
curl에서 index 조회·bulk 입력·query 조회 모두 동일 헤더로 인증합니다.

---

## 2. API Key 생성

```json
POST /_security/api_key
{
  "name": "prometheus-exporter",
  "role_descriptors": {
    "monitoring_role": {
      "cluster": ["monitor"],
      "index": [
        { "names": ["*"], "privileges": ["monitor", "view_index_metadata"] }
      ]
    }
  }
}
```
응답:
```json
{
  "id": "8hzCO5gB8jLSlg7g-LkS",
  "api_key": "AK_5px10QRajXigvwVzQyQ",
  "encoded": "OGh6...UQ=="   // ← 클라이언트가 사용하는 API key
}
```

## 3. API Key 사용 (curl)

```bash
export ES_URL="https://log-es1:9200"
export API_KEY="TW9iWkw1Z0...Q3Vw=="

# index 정보
curl --cacert ../cert/http_ca.crt -X GET "${ES_URL}/bos.useractlog" \
  -H "Authorization: ApiKey ${API_KEY}" -H "Content-Type: application/json" | jq

# bulk 입력
curl -X POST "${ES_URL}/_bulk?pretty" \
  -H "Authorization: ApiKey ${API_KEY}" -H "Content-Type: application/json" -d'
{ "index": { "_index": "index_name" } }
{"name":"1984","author":"George Orwell","page_count":328}
'

# query 조회
curl -X POST "${ES_URL}/index_name/_search?pretty" \
  -H "Authorization: ApiKey ${API_KEY}" -H "Content-Type: application/json" -d'
{ "query": { "query_string": { "query": "snow" } } }'
```

> HTTPS(`https://`) 사용 시 `--cacert`로 CA 인증서 지정.

## 4. 연관 개념

- [[2026-06-14-ES05_(Elasticsearch-Dev-Tools-DSL-조회)]]
- [[2026-06-14-K04_(Kafka-Connector-인증서-truststore-등록)]]
