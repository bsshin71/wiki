# Elasticsearch Dev Tools DSL 조회 (클러스터·샤드·alias)

- **카테고리**: #elasticsearch #admin
- **태그**: #elasticsearch #DSL #devtools #cat_api #cluster_health
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-DEV TOOL - BigDataTeam]]

## 1. 핵심 요약

Kibana **Dev Tools**에서 REST API/DSL로 클러스터 상태(`_cluster/health`), 샤드 현황(`_cat/shards`), alias(`_cat/aliases`)를 조회합니다.
`_cat/*` API는 사람이 읽기 좋은 표 형식이며, `v`(헤더)·`s`(정렬)·`h`(컬럼)·`bytes` 파라미터로 출력을 제어합니다.

---

## 2. 클러스터 상태

```
GET _cluster/health
# status(green/yellow/red), number_of_nodes, active_primary_shards,
# active_shards, unassigned_shards, active_shards_percent_as_number
```
> `unassigned_shards > 0`이면 샤드 미할당(노드 부족/디스크 부족 등) 점검 필요. green=정상.

## 3. 샤드 조회 (_cat/shards)

```
# 특정 인덱스 샤드 (prirep: p=primary, r=replica)
GET _cat/shards/bosactlog?v

# 전체 샤드, 크기순 정렬(GB)
GET _cat/shards?v=true&h=index,prirep,shard,store&s=prirep,store&bytes=gb

# 각 노드별 샤드/크기
GET _cat/shards?v
```

## 4. 인덱스 / alias 조회

```
GET _cat/indices?v&s=index    # 인덱스별 상태·용량
GET _cat/aliases              # alias → 실제 인덱스 매핑 (rollover write alias 등)
GET _cat/nodes?v              # 노드 목록
```

## 5. 연관 개념

- [[2026-06-14-ES01_(Elasticsearch-시스템-구조-클러스터-노드-샤드)]]
- [[2026-06-14-ES03_(Elasticsearch-Cluster-Shard-수-확인-및-증가)]]
- [[2026-06-14-ES04_(Elasticsearch-API-Key-연결)]]
