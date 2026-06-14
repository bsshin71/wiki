# Elasticsearch 시스템 구조 (클러스터·노드·샤드)

- **카테고리**: #elasticsearch #구조
- **태그**: #elasticsearch #cluster #node #shard #discovery
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Elasticsearch 시스템 구조 - BigDataTeam]]

## 1. 핵심 요약

Elasticsearch의 최상위 단위는 **Cluster**이며, 같은 `cluster.name`을 가진 노드들이 자동 바인딩되어 하나의 클러스터를 이룹니다.
데이터는 **샤드(Primary)+복사본(Replica)** 으로 분할 저장되고, 노드 탐색은 **디스커버리(유니캐스트 권장)** 로 이루어집니다.

---

## 2. 클러스터와 노드

- 클러스터 = 1개 이상의 노드. 서로 다른 클러스터는 데이터 접근/교환 불가(독립).
- 노드 = 하나의 ES 프로세스. 같은 `cluster.name`으로 실행 시 **자동 바인딩**(9300 데이터 통신 포트).
- `cluster.name`이 다르면 별도 클러스터 구성(같은 서버에서 http 9202, data 9302로 충돌 없이 실행).

### 마스터/데이터 노드
| 노드 | 역할 |
|------|------|
| **마스터** (`node.master: true`) | 클러스터 상태 메타 정보 관리, 종료 시 재선출 |
| **데이터** (`node.data`) | 색인 데이터 실제 저장 (false면 미저장) |

## 3. 샤드와 복사본

- 색인 데이터는 여러 샤드로 분할. (구버전 기본 5 샤드 + 복사본)
- **Primary Shard**: 최초 색인 저장 공간. 이후 동일 수의 **Replica** 생성.
- Primary와 Replica는 **서로 다른 노드**에 저장 → 노드 2개 이상 유지 권장(무결성).
- 예: 5GB·5샤드·1복사본 → 샤드당 1GB, 총 10샤드/10GB.
- 설정: `index.number_of_shards`, `index.number_of_replicas`.

## 4. 네트워크 바인딩 & 디스커버리

- 노드 간 통신 포트 9300~ 개방 시 네트워크 바인딩 가능.
- **젠 디스커버리**: 멀티캐스트(자동 검색, 기본) vs **유니캐스트(주소 지정, 권장)**.
  - 멀티캐스트는 의도치 않은 바인딩·불안정 → 유니캐스트 권장.
- 방화벽 내부↔외부 바인딩 시 네트워크 호스트 설정:
  ```yaml
  network.bind_host: 192.168.111.153   # 내부 주소
  network.publish_host: 121.131.44.50  # 인터넷 주소
  ```
- 바인딩 조건: **동일 cluster.name + 동일 버전 + 9300 포트 개방**.

## 5. 연관 개념

- [[2026-06-14-ES02_(Elasticsearch-Cluster-구축-아키텍처)]]
- [[2026-06-14-ES03_(Elasticsearch-Cluster-Shard-수-확인-및-증가)]]
- [[2026-06-14-ES05_(Elasticsearch-Dev-Tools-DSL-조회)]]
