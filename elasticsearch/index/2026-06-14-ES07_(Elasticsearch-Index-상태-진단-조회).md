# Elasticsearch Index 상태/진단 조회

- **카테고리**: #elasticsearch #admin
- **태그**: #elasticsearch #index #shard #allocation #ISM #진단
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-특정 인덱스 상태 조회 - BigDataTeam]], [[2026-06-14-index 상태 조회하기 - BigDataTeam]]

## 1. 핵심 요약

특정 인덱스의 상태는 `_cat/indices`(health·docs)·`_cat/shards`(샤드별 상태)로 확인하고, **UNASSIGNED 샤드 원인은 `_cluster/allocation/explain`** 으로 진단합니다.
ISM(Index State Management) 정책 진행 상태는 `_opendistro/_ism/explain/<인덱스>`로 확인합니다.

---

## 2. 인덱스 / 샤드 상태

```
GET _cat/indices/swz-backuplog-2025.11.30?v
# health(yellow=replica 미할당), docs.count, store.size

GET _cat/shards/swz-backuplog-2025.11.30?v
# prirep(p/r), state(STARTED/UNASSIGNED), node
```
> replica가 `UNASSIGNED`면 health=yellow. 단일 노드에서 replica 미할당이 대표 원인.

## 3. 샤드 미할당 원인 진단 (allocation explain)

```
GET _cluster/allocation/explain      # 미지정 시 임의 unassigned 샤드 설명
# 특정 샤드:
# { "index":"...", "shard":0, "primary":false }
```
핵심 해석:
- `unassigned_info.reason`: INDEX_CREATED / NODE_LEFT / ALLOCATION_FAILED 등
- `node_allocation_decisions[].deciders`: 할당 거부 이유
  - 예: `same_shard` decision NO → **primary와 동일 노드에 replica 배치 불가**(노드 부족) → 노드 증설 또는 replica 0 조정

## 4. ISM 정책 상태 (OpenDistro)

```
GET _opendistro/_ism/explain/test-index-000001
# policy_id, rolled_over(true/false), policy_completed, enabled, policy_seq_no
```
> ISM 관리 인덱스의 rollover 여부·정책 완료 상태 확인.

## 5. 연관 개념

- [[2026-06-14-ES03_(Elasticsearch-Cluster-Shard-수-확인-및-증가)]]
- [[2026-06-14-ES05_(Elasticsearch-Dev-Tools-DSL-조회)]]
- [[2026-06-14-ES09_(Elasticsearch-인덱스-구성-방안-ISM-Policy)]]
