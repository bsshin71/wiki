# PostgreSQL Patroni HA 구성

- **카테고리**: #DBMS #PostgreSQL #replication
- **태그**: #PostgreSQL #HA #patroni #etcd #haproxy #failover
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Patroni - BigDataTeam]]

## 1. 핵심 요약

Patroni는 PostgreSQL **HA 구성·자동 failover**를 관리하는 도구로, 클러스터 상태를 **etcd(key-value store)** 에 저장해 동작하고 **HAProxy**로 부하 분산/접속 라우팅을 합니다.
성숙·안정적이어서 Percona·Google Cloud의 PG HA 표준으로 쓰이며, **etcd·HAProxy 자체의 이중화**도 함께 고려해야 합니다.

---

## 2. 구성 요소

| 요소 | 역할 |
|------|------|
| **Patroni** | PG HA 구축·유지보수·자동 failover 관리(etcd 의존) |
| **etcd** | PG 상태 저장 key-value store. Patroni가 기반으로 동작 |
| **HAProxy** | load balancer (primary 접속 라우팅) |

## 3. 특징

- 프로젝트 성숙·안정성 검증.
- **Percona / Google Cloud**의 PostgreSQL HA 표준 구성으로 채택.
  - https://docs.percona.com/postgresql/16/solutions/ha-setup-yum.html

## 4. 고려 사항

- **etcd 이중화**: etcd도 store이므로 이중화 구성 필요, **PG 서버와 물리적 분리** 권장.
- **HAProxy 이중화** 구성 고려(SPOF 방지).

## 5. 연관 개념

- [[2026-06-14-PG15_(PostgreSQL-복제-방식-물리-논리)]]
- [[2026-06-14-PG16_(PostgreSQL-복제-구성-실습-Streaming-Logical)]]
- [[2026-06-14-65_(MySQL-MHA-설치-RedHat8)]]
