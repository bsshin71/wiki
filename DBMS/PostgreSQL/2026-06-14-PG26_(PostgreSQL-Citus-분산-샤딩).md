# PostgreSQL Citus 분산 샤딩

- **카테고리**: #DBMS #PostgreSQL #구조
- **태그**: #PostgreSQL #citus #sharding #scale-out #distributed
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Citus 를 활용한 DB 샤딩 - BigDataTeam]]

## 1. 핵심 요약

Citus는 PostgreSQL **scale-out 확장**으로, 테이블을 **hash 기반으로 워커 노드에 분산**합니다.
**Coordinator**(메타데이터·쿼리 라우팅·최종 정렬) + **Worker node**(실제 데이터 처리)로 구성되며, 노드 간 HA·백업은 별도로 구축해야 합니다.

---

## 2. 특징

- PostgreSQL scale-out extension (MySQL의 plugin 격).
- **hash 기반 테이블 분산**.
- 엔터프라이즈 버전: 자동 리밸런싱 + 커넥션 풀 지원.
- ⚠️ 노드 간 **HA 별도 구축** 필요, **별도 백업 정책** 필요.
- 아키텍처가 MongoDB sharding + MySQL spider engine과 유사.

## 3. 구성 요소

| 역할 | 설명 |
|------|------|
| **Coordinator** | distributed table 메타데이터 관리, 클라이언트 요청을 워커에 전달, **쿼리 최종 정렬** 수행 |
| **Worker node** | coordinator 요청 받아 실제 데이터 처리, DML/DDL/ANALYZE/VACUUM 등 명령 실행 후 결과 전달 |

## 4. 연관 개념

- [[2026-06-14-PG02_(PostgreSQL-아키텍처-및-특징)]]
- [[2026-06-14-PG13_(PostgreSQL-Online-파티션-테이블-재구성)]]
- [[2026-06-14-PG18_(PostgreSQL-Patroni-HA-구성)]]
