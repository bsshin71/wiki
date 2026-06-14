# Elasticsearch 운영 오류 (Primary Shard not active)

- **카테고리**: #elasticsearch #오류
- **태그**: #elasticsearch #troubleshooting #shard #allocation #참고자료
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-ElasticSearch 오류 - BigDataTeam]], [[2026-06-14-Elasticsearch 참고자료 - BigDataTeam]]

## 1. 핵심 요약

`primary shard is not active`(unavailable_shards_exception)는 bulk/색인 시 대상 primary 샤드가 미할당/비활성 상태일 때 발생합니다.
**`_cluster/allocation/explain`** 으로 해당 샤드의 미할당 원인을 진단해 해결합니다.

---

## 2. 오류: primary shard is not active

```
unavailable_shards_exception, reason=[devboslog-bos.trade.pc-20210520][3]
  primary shard is not active Timeout: [1m], request: [BulkShardRequest ...]
```
> bulk 색인 중 대상 primary 샤드가 active가 아니어서 1분 타임아웃.

## 3. 진단: allocation explain

```json
GET /_cluster/allocation/explain
{ "index": "devboslog-bos.trade.pc-20210520", "shard": 1, "primary": true }
```
- `unassigned_info.reason` / `deciders`로 미할당 원인 확인.
- 흔한 원인: 노드 이탈, 디스크 watermark 초과, allocation 설정 제약.
- 조치: 노드/디스크 확보, allocation 설정 점검, 필요 시 `_cluster/reroute`.

> 참고: [Primary shard not available](https://discuss.elastic.co/t/primary-shard-not-available/161687)

## 4. 추가 학습 자료 (PDF)

> `2026-06-14-Elasticsearch 참고자료`의 첨부 PDF — 단독 wiki 문서 가치가 낮아 본 문서에 자료 링크로 통합.

- **엘라스틱서치 Aggregation 고급** — `13-ElasticSearch-Aggs2.pdf`
- **ElasticSearch 작용 및 활용** — `20150317elasticsearch-...-gate01.pdf`

## 5. 연관 개념

- [[2026-06-14-ES07_(Elasticsearch-Index-상태-진단-조회)]]
- [[2026-06-14-ES03_(Elasticsearch-Cluster-Shard-수-확인-및-증가)]]
