# Elasticsearch 개요 (허브)

- **카테고리**: #elasticsearch #허브
- **태그**: #elasticsearch #허브 #목차 #overview
- **작성일**: 2026-06-14

## 1. 핵심 요약

> 이 문서는 **elasticsearch 카테고리의 탐색 진입점(허브)** 입니다. 21개 문서를 `cluster`·`index`·`query` sub 폴더로 분류해 목차로 제공합니다.
> 클러스터·구조(cluster) → 인덱스·라이프사이클(index) → 검색·쿼리(query) 순으로 보면 됩니다.

---

## 2. 🗄️ cluster/ — 클러스터 · 구조 · 연결 · 운영

- [[2026-06-14-ES01_(Elasticsearch-시스템-구조-클러스터-노드-샤드)]] — 클러스터/노드/샤드 기본
- [[2026-06-14-ES02_(Elasticsearch-Cluster-구축-아키텍처)]] — 마스터/데이터 분리·Hot-Warm
- [[2026-06-14-ES03_(Elasticsearch-Cluster-Shard-수-확인-및-증가)]] — shard 상한 관리
- [[2026-06-14-ES04_(Elasticsearch-API-Key-연결)]] — API Key 인증
- [[2026-06-14-ES05_(Elasticsearch-Dev-Tools-DSL-조회)]] — Kibana Dev Tools
- [[2026-06-14-ES19_(Elasticsearch-ElastAlert-알림-도구)]] — ElastAlert 알림
- [[2026-06-14-ES21_(Elasticsearch-운영-오류-Primary-Shard-not-active)]] — 샤드 미할당 진단

## 3. 📑 index/ — 인덱스 관리 · ISM · Rollover

- [[2026-06-14-ES06_(Elasticsearch-Index-관련-명령-매핑-Field삭제-Rollover)]]
- [[2026-06-14-ES07_(Elasticsearch-Index-상태-진단-조회)]]
- [[2026-06-14-ES08_(Elasticsearch-Index-Rename-reindex)]]
- [[2026-06-14-ES09_(Elasticsearch-인덱스-구성-방안-ISM-Policy)]]
- [[2026-06-14-ES10_(Elasticsearch-Hot-Warm-Delete-ISM-Rollover-구성)]]
- [[2026-06-14-ES11_(Elasticsearch-Rollover-Alias-개념)]]
- [[2026-06-14-ES12_(Elasticsearch-ISM-Policy-Update-동시성제어)]]
- [[2026-06-14-ES13_(Elasticsearch-Index명-날짜함수-Date-Math)]]
- [[2026-06-14-ES14_(Elasticsearch-ISM-History-비활성화-및-정리)]]
- [[2026-06-14-ES20_(Elasticsearch-테스트용-Index-구성-실습-Rollover-Ngram)]]

## 4. 🔎 query/ — 검색 · 쿼리

- [[2026-06-14-ES15_(Elasticsearch-Like-검색-Wildcard-Ngram)]]
- [[2026-06-14-ES16_(Elasticsearch-Nested-Object-Array-조회)]]
- [[2026-06-14-ES17_(Elasticsearch-조건-filtering-update_by_query)]]
- [[2026-06-14-ES18_(Elasticsearch-인덱스-생성-데이터-추가-조회-기본)]]

## 5. 연관 개념 (타 카테고리 연결)

- [[2026-06-14-K11_(Kafka-Elasticsearch-Sink-Connector)]] — Kafka→ES 적재 파이프라인
- [[2026-06-14-K18_(Kafka-모니터링-Prometheus-Alertmanager)]] — 알림 도구(ElastAlert와 비교)
- [[2026-06-14-Kafka-개요-허브]] · [[2026-06-14-PostgreSQL-개요-허브]]
