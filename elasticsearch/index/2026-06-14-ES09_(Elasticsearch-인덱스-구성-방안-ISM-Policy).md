# Elasticsearch 인덱스 구성 방안 (ISM Policy)

- **카테고리**: #elasticsearch #admin
- **태그**: #elasticsearch #인덱스구성 #ISM #policy #hot-warm #rollover
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-elasticsearch 인덱스 구성 - BigDataTeam]]

## 1. 핵심 요약

인덱스 구성은 **① 날짜별 rotation(devboslog-YYYYMMDD)** vs **② 단일 index명 고정 + rollover** 두 방안이 있습니다.
2안은 매핑(nested array)·shard 수·Hot-Warm 라이프사이클을 제어할 수 있어 권장되며, **ISM Policy**로 hot→cold 상태 전이를 자동화합니다.

---

## 2. 인덱스 구성 두 방안

| 방안 | index명 | 장점 | 단점 |
|------|---------|------|------|
| **1안 timestamp rotation** | `devboslog-YYYYMMDD` | 날짜별 삭제·유지관리 쉬움, 스키마 변경 대응 유리 | sink connector 자동생성 → **nested array 설정 불가**, shard 수 예측 어려움 |
| **2안 단일 index 고정** | `devboslog` | 사전 매핑(object array nested) 설정 가능, **적정 shard 조정**, Hot-warm-cold-delete 라이프사이클 적용 | 오래된 데이터 이동 작업 필요, 스키마 변경 시 재생성, **rollover 패턴 필요** |

> nested array 검색·Hot-Warm이 필요하면 **2안(단일 index + rollover + ISM)** 권장.

## 3. ISM Policy 생성 (OpenDistro)

> ⚠️ ES 버전별 syntax 상이 → 버전 매뉴얼 참고.

```json
{
  "policy": {
    "default_state": "hot",
    "states": [
      { "name": "hot",
        "actions": [ { "replica_count": { "number_of_replicas": 5 } } ],
        "transitions": [ { "state_name": "cold", "conditions": { "min_index_age": "30d" } } ] },
      { "name": "cold",
        "actions": [ { "replica_count": { "number_of_replicas": 2 } } ],
        "transitions": [] }
    ]
  }
}
```
> hot(replica 5) → 30일 경과 → cold(replica 2) 전이. Hot-Warm-Cold-Delete 라이프사이클을 상태/전이로 정의.

## 4. 연관 개념

- [[2026-06-14-ES02_(Elasticsearch-Cluster-구축-아키텍처)]]
- [[2026-06-14-ES06_(Elasticsearch-Index-관련-명령-매핑-Field삭제-Rollover)]]
- [[2026-06-14-ES07_(Elasticsearch-Index-상태-진단-조회)]]
