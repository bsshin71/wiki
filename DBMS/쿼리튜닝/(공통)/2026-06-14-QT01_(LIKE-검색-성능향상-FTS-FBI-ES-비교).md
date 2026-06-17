# LIKE '%검색어%' 성능 향상 방안 비교 (FTS·FBI·ES)

- **카테고리**: #DBMS #쿼리튜닝
- **태그**: #쿼리튜닝 #LIKE #full-text-search #FBI #elasticsearch #인덱스
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Like '%검색어%' 성능 향상 방안 비교 - BigDataTeam]]

## 1. 핵심 요약

`LIKE '%검색어%'`(앞 % 포함)는 인덱스가 있어도 전체 스캔이 발생한다. 향상 방안 3가지: ① **ngram Full-Text Search** (MySQL 내장, 쓰기 성능↓), ② **명시적 생성 칼럼+FBI 역방향 인덱스** (`%검색어`만 가능, `%검색어%`는 불가), ③ **Elasticsearch** (최고 성능·고급 기능, 동기화 필요).

---

## 2. 방식별 비교표

| 방식 | `%검색어%` 성능 | `%검색어` 성능 | 쓰기 성능 | 저장공간 | 단점 | 장점 |
|------|----------------|----------------|-----------|----------|------|------|
| **1) ngram FTS** | 좋음 | 좋음 | ↓ 하락 | 인덱스 사이즈 대폭 증가 | 쓰기 성능↓, 백업 크기↑, 코드 변경 필요 | DB 단독 구현, `%키워드%` 지원 |
| **2) 생성칼럼+FBI** | ❌ 불가(full scan) | 좋음 | 미미한 영향 | index 공간만(virtual) | `%검색어%` 처리 불가 | 구현 단순 |
| **3) Elasticsearch** | 최고 | 최고 | N/A | N/A | MySQL↔ES 동기화 필요 | 대용량·고급검색 |

## 3. 선택 가이드

- `%검색어%` 양측 LIKE + MySQL 단독: **ngram FTS** (인덱스 비대화·쓰기 성능 감수)
- `%검색어` 후방 LIKE만: **생성 칼럼+FBI** (REVERSE 함수 + virtual column + index)
- 대용량·고품질 검색: **Elasticsearch** (Kafka Connector 또는 Logstash 동기화 병행)

## 4. 연관 개념

- [[2026-06-14-MS08_(MySQL-Full-Text-Search-MATCH-AGAINST-ngram)]] — ngram FTS 상세
- [[2026-06-14-MS09_(MySQL-Ngram-Full-Text-Search-장단점)]] — ngram 장단점
- [[2026-06-14-MS06_(MySQL-Functional-Index-FBI-LIKE-한계)]] — FBI+LIKE 버그
- [[2026-06-14-MS07_(MySQL-Generated-Column-Virtual-Stored-인덱스)]] — 생성칼럼+인덱스
- [[2026-06-14-Elasticsearch-개요-허브]] — ES 대안
