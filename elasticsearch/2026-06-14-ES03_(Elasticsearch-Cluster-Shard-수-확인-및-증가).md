# Elasticsearch Cluster Shard 수 확인 및 증가

- **카테고리**: #elasticsearch #admin
- **태그**: #elasticsearch #shard #cluster #장애 #max_shards_per_node
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-cluster shard 수 확인 및 증가 - BigDataTeam]]

## 1. 핵심 요약

클러스터 전체 shard 상한 = **노드 수 × `cluster.max_shards_per_node`(기본 1000)**.
상한 초과 시 인덱스 생성이 실패(`this action would add [X] total shards, but ... maximum shards open`)하므로 `max_shards_per_node`를 상향하거나 노드를 증설합니다.

---

## 2. 노드당 최대 shard 설정 확인

```
GET /_cluster/settings?include_defaults=true
# 미변경: "defaults.cluster.max_shards_per_node" (기본 1000)
# 변경됨: "persistent.cluster.max_shards_per_node"
```

## 3. 클러스터 전체 최대 shard 계산

```
전체 최대 shard = 노드 수 × cluster.max_shards_per_node
GET /_cat/nodes?v        # 노드 수 확인
```

## 4. 현재 shard 개수 확인

```
GET /_cluster/health?pretty       # active_primary_shards 확인
GET /_cat/indices?v&s=index       # 인덱스별 shard 수
```

## 5. 최대값 변경 (필요 시)

```
PUT /_cluster/settings
{
  "persistent": { "cluster.max_shards_per_node": 1500 }
}
```

## 6. 상한 초과 오류

```
this action would add [X] total shards, but this cluster currently has [Y]/[Z] maximum shards open
```
→ `max_shards_per_node` 상향 또는 노드 증설, 불필요 인덱스 삭제.

## 7. 연관 개념

- [[2026-06-14-ES01_(Elasticsearch-시스템-구조-클러스터-노드-샤드)]]
- [[2026-06-14-ES05_(Elasticsearch-Dev-Tools-DSL-조회)]]
