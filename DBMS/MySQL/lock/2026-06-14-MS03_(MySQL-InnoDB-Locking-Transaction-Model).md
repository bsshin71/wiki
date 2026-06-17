# MySQL InnoDB Locking & Transaction Model

- **카테고리**: #DBMS #MySQL #lock
- **태그**: #MySQL #lock #innodb #transaction #isolation #gap-lock #next-key #deadlock
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-InnoDB Locking and Transaction Model - BigDataTeam]]

## 1. 핵심 요약

MySQL 락은 **서버 레벨(global·table·user·metadata)** 과 **InnoDB 레벨(S/X·Intention·Record·Gap·Next-Key·Insert-Intention·AUTO-INC)** 로 나뉜다. 트랜잭션 격리수준(RU/RC/RR/SER)에 따라 gap lock 동작이 달라지며, **RR**에서 Next-Key 락으로 phantom을 방지하고 **RC**에서는 gap lock이 비활성화된다. 이 문서는 개념 레퍼런스(이론·예시 중심).

---

## 2. 서버 레벨 락

| 락 | 획득 | 비고 |
|----|------|------|
| global | `FLUSH TABLES WITH READ LOCK` | 서버 전체, 백업용 |
| table | `LOCK TABLES t [READ|WRITE]` | MyISAM/MEMORY 묵시적 |
| user(named) | `GET_LOCK('s',n)/IS_FREE_LOCK/RELEASE_LOCK` | 문자열 대상, 앱 동기화 |
| metadata(MDL) | 자동 | table/schema/stored program/tablespace, 쓰기 우선(`max_write_lock_count`) |

## 3. InnoDB 레벨 락

- **S/X**: 공유(읽기)/배타(쓰기) 행 락.
- **Intention(IS/IX)**: 테이블+행 다중 세분화. 호환표 — X는 모두 충돌, IX/IS는 상호 호환.
- **Record Lock**: 인덱스 레코드 락(인덱스 없으면 GEN_CLUST_INDEX).
- **Gap Lock**: 인덱스 레코드 사이 간격 잠금(INSERT 제어). RC에서 비활성.
- **Next-Key**: Record+Gap 조합, RR phantom 방지, secondary index에서 발생.
- **Insert Intention**: INSERT 직전 gap 락, 위치 다르면 대기 없음.
- **AUTO-INC**: `innodb_autoinc_lock_mode` 0(traditional)/1(consecutive, 5.7 기본)/2(interleaved, 8.0 기본·table lock 회피). 모드 2는 SQL 기반 복제 시 비안전·갭 가능.

## 4. 트랜잭션 격리수준

| 수준 | 특징 |
|------|------|
| READ UNCOMMITTED | dirty read 가능 |
| READ COMMITTED | gap lock 비활성 → phantom 가능 |
| **REPEATABLE READ**(기본) | 첫 read 스냅샷 유지, gap/next-key로 phantom 방지 |
| SERIALIZABLE | SELECT가 모두 `FOR SHARE`로 실행 |

## 5. Deadlock

- `innodb_deadlock_detect` on/off, 감지 시 자동 rollback.
- `SHOW ENGINE INNODB STATUS`(최신만), `innodb_print_all_deadlocks`로 전체 로그.

## 6. 연관 개념

- [[2026-06-14-MS04_(MySQL-InnoDB-스토리지엔진-잠금-레코드-갭-넥스트키)]]
- [[2026-06-14-MS05_(MySQL-엔진-잠금-Global-Table-Named-Metadata)]]
- [[2026-06-13-16_(MySQL-Lock-개념-및-분석)]] · [[2026-06-13-23_(MySQL-Deadlock-분석-및-해결)]]
