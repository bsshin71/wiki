# Lint Report — 2026-06-14

## 요약
- 점검 문서 수: 168개 파일 (~63,244 단어)
- 그래프 기준: 1713 노드 · 1545 엣지 · 168 커뮤니티 (2026-06-14 reorg 후 재빌드)
- 발견된 이슈: **14건** (P1 1 · P2 6 · P3 7)
- 핵심: ① Sub 폴더 정리 정상 완료 검증 ② **신규 카테고리(kafka·ES·PG) 간/기존 문서와의 교차참조 단절(island)** ③ kafka·ES·PG **허브(목차) 문서 부재**

---

## ✅ Sub 폴더 정리 검증 (사용자 요청 1)

야간 배치로 root에 집중됐던 문서를 주제별 sub 폴더로 이동 완료. **모든 카테고리 root 집중 해소.**

| 카테고리 | 이전(root) | 정리 후 |
|----------|:---:|------|
| kafka/ | 18 | connect(9)·core(5)·tools(4), root 0 |
| elasticsearch/ | 21 | cluster(7)·index(10)·query(4), root 0 |
| DBMS/PostgreSQL/ | 31 | install(4)·admin(11)·monitoring(5)·replication(3)·maintenance(6), root 5(misc) |
| DBMS/MySQL/ | (정상) | replication·backup·lock·admin·installation·innodb·ha (기존 정상) |

- index.md 경로 필드 **66건 자동 갱신**(파일시스템 기준), 전체 카테고리 표 sub 폴더 행 추가.
- 백링크는 Obsidian 위키링크(`[[명]]`)라 경로 무관 → **깨진 링크 0**.

---

## 자동 발견 (graphify)

### 🌟 God Nodes (연결 상위)
1. `Percona MySQL 운영(PRD) 표준 my.cnf` — 16
2. `MySQL GTID 기반 복제` — 15
3. `MySQL Semi-Synchronous 복제` — 15
4. `MySQL Replication 명령어 모음` — 14
5. `MySQL Performance Schema 활용` — 14
6~10. MySQL Binary Log·Admin 쿼리·Percona 설치·auto_increment 등

> ⚠️ God Node 상위가 **전부 MySQL**. 신규 kafka·ES·PG는 각자 내부 연결만 형성하고 상위 허브에 진입하지 못함 → 카테고리 간 단절(아래 P2).

### 구조 점검
- **Import Cycle**: 없음 · **Surprising/모순 연결**: 없음.
- **isolated node 1231개**: 대부분 문서 내부 섹션 헤더(`1. 핵심 요약` 등) — 정상 노이즈. 단, 일부 신규 leaf 문서는 인바운드 0(아래 P2).

---

## 분석 발견 (Claude)

### 🔴 P1 — kafka·ES·PG 허브(목차) 문서 부재
MySQL은 5개 허브 문서(Lock·Replication·XtraBackup·Percona·관리자)로 탐색 진입점이 있으나, **신규 3개 카테고리는 허브가 없음**.
- 권장: 각 카테고리에 `_(Kafka-개요-허브)` / `_(Elasticsearch-개요-허브)` / `_(PostgreSQL-개요-허브)` 생성 → sub 폴더별 문서 링크 목록 제공(MySQL 허브 방식 재사용).

### 🟠 P2 — 카테고리 간 교차참조 단절 (island)
의미상 연결되지만 백링크가 없는 문서 쌍(브리지 추가 권장):

- [[2026-06-14-K02_(Kafka-Connect-설정-Debezium-CDC)]] ↔ [[2026-06-13-01_(MySQL-Binary-Log-Position-복제)]] / [[2026-06-13-20_(MySQL-Binary-Log-활성화-비활성화)]] — Debezium은 MySQL binlog CDC, 상호 링크 권장
- [[2026-06-14-K11_(Kafka-Elasticsearch-Sink-Connector)]] ↔ [[2026-06-14-ES18_(Elasticsearch-인덱스-생성-데이터-추가-조회-기본)]] — Kafka→ES 적재 파이프라인 연결
- [[2026-06-14-K04_(Kafka-Connector-인증서-truststore-등록)]] ↔ [[2026-06-14-ES04_(Elasticsearch-API-Key-연결)]] — ES 연동 인증
- [[2026-06-14-PG23_(PostgreSQL-mysql_fdw-MySQL-연동)]] ↔ [[2026-06-02_(MySQL-데이터이관-Shell-스크립트)]] — 이미 일부 링크됨(정상)
- [[2026-06-14-K18_(Kafka-모니터링-Prometheus-Alertmanager)]] ↔ [[2026-06-14-ES19_(Elasticsearch-ElastAlert-알림-도구)]] — 알림 도구 비교 링크(일부 연결됨)

### ⚪ P2 — 인바운드 0(고립) 신규 leaf 문서
다른 문서가 가리키지 않아 탐색 고립 위험:
- [[2026-06-14-PG26_(PostgreSQL-Citus-분산-샤딩)]] — 인바운드 0
- [[2026-06-14-PG14_(PostgreSQL-pg_dump-테이블-DDL-추출)]] — PG16/PG13에서만 약하게 연결
- [[2026-06-14-K15_(Kafka-성능-튜닝-커널-디스크-GC)]] — PG63(커널)과만 연결
- [[2026-06-14-ES21_(Elasticsearch-운영-오류-Primary-Shard-not-active)]] — ES03/ES07에서 역링크 보강 권장

### 🟠 P2 — MySQL 허브 God Node 잔존(이전 lint 후속)
2026-06-13 lint에서 허브화한 5개 통합문서가 여전히 God Node 상위. 의도된 설계(탐색 허브)이므로 **유지**. 단 개별 문서 신규 추가 시 허브 목록 갱신 필요.

### 🟡 P3 — 잠재적 내용 인접/중복
- [[2026-06-14-PG08_(PostgreSQL-세션-관리-연결제어)]] ↔ [[2026-06-14-PG22_(PostgreSQL-too-many-connections-오류)]] — 연결 종료 쿼리 일부 중복(상호 링크됨, 의도적 분리 OK)
- [[2026-06-14-PG17_(PostgreSQL-AutoVacuum-설정-및-튜닝)]] ↔ [[2026-06-14-PG27_(PostgreSQL-Vacuum-개념-MVCC-XID-Wraparound-Freeze)]] — 개념/튜닝 분리(정상, 상호 링크됨)
- [[2026-06-14-K06_(Kafka-설치-Apache-Confluent-Debezium-CDC)]] ↔ [[2026-06-14-K01_(Kafka-Connect-설치-confluent-hub)]] — confluent-hub 설치 일부 중복(관점 차이로 분리 유지)

### 🟡 P3 — 낡은 주장
- 해당 없음(전 문서 2026-06 작성). 단 원본이 2019~2025 혼재 → 버전 명시 권장(예: PG 소스설치 RHEL 버전 의존성, Kafka/ES 버전).

### 🔵 P3 — 자체 페이지 없이 다수 언급 개념
- **Debezium** — K01·K02·K06 등에서 반복. K02가 사실상 대표(추가 생성 불필요).
- **WAL / pg_stat_activity** — 복제·모니터링 다수 문서에서 언급, PG15/PG19가 주 설명(정상).

---

## 권장 조치 (우선순위)
1. **(P1) kafka·ES·PG 허브 문서 3개 생성** — sub 폴더별 문서 링크 목록(MySQL 허브 패턴).
2. **(P2) 교차참조 브리지 추가** — K02↔MySQL binlog, K11↔ES18, K04↔ES04 등 파이프라인 연결.
3. **(P2) 고립 leaf 역링크 보강** — PG26/PG14/K15/ES21에 인바운드 추가.
4. **(P3) 버전 주석 보강** — 원본 연도 차이 큰 문서에 버전 경계 명시.

> ⚠️ 본 보고서는 점검 결과만 기록하며 문서를 수정하지 않았습니다. 위 조치는 승인 후 진행합니다.
