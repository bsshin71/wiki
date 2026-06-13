# PostgreSQL 아키텍처 및 특징

- **카테고리**: #DBMS #PostgreSQL #구조
- **태그**: #PostgreSQL #아키텍처 #MVCC #vacuum #MySQL비교
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-Postgres 아키텍쳐 및 특징 - BigDataTeam]]

## 1. 핵심 요약

PostgreSQL은 **MVCC 기반 ACID 트랜잭션 DB**로, 분석 기반 시스템에 강점(HASH/SORT 조인, 병렬쿼리, with절, GIS)이 있습니다.
MVCC 특성상 **dead tuple이 쌓여 주기적 Vacuum이 필수**이며, 이를 게을리하면 Disk I/O 증가로 성능이 저하됩니다.

---

## 2. 용어 (Oracle/산업 용어 → PostgreSQL)

| 산업 용어 | PostgreSQL |
|-----------|-----------|
| Table/Index | Relation |
| Row | Tuple |
| Column | Attribute |
| Data Block(disk) | Page |
| Page(memory) | Buffer |

## 3. 주요 Limits

| 항목 | 값 |
|------|----|
| Database Size | Unlimited |
| Relation Size | 32TB |
| Columns per Table | 1600 |
| Field Size | 1GB |
| Identifier Length | 63 bytes |
| Partition Keys / Columns per Index | 32 |

## 4. MySQL vs PostgreSQL (2019 기준, 참고)

| 항목 | MySQL | PostgreSQL |
|------|-------|-----------|
| online DDL | 많이 지원 | 대부분 미지원 |
| UPDATE | 빠름(in-place) | 느림(out-of-place) |
| DELETE | 느림 | 빠름 |
| JOIN(HASH/SORT) | 제한적(8.0+) | 지원 |
| 기본 격리수준 | repeatable-read | read-committed |
| with절/병렬쿼리/GIS | 제한적 | 지원 |
| 적합 업무 | Web 서비스 | 분석 기반 시스템 |

## 5. 단점 — Vacuum 필요 (MVCC)

- MVCC는 old version(dead tuple)을 즉시 삭제하지 않고 보관.
- UPDATE/DELETE 多 → dead tuple 증가 → Disk I/O 증가 → 성능 저하.
- **Vacuum으로 주기적으로 dead tuple 정리** 필요.

## 6. 주요 특징

- ACID·트랜잭션, **MVCC**(동시성), 다양한 인덱싱, Full-text search.
- 유연한 복제 방식, 다양한 PL 언어(PL/pgSQL·Python·Perl…), 인터페이스(JDBC/ODBC…).
- GIS(PostGIS), Key-Value(HStore), DBLink, Crypto/UUID 함수.

## 7. 연관 개념

- [[2026-06-14-PG01_(PostgreSQL-소스-컴파일-설치)]]
- [[2026-06-14-PG05_(PostgreSQL-사용자-Role-관리)]]
