# MySQL Replication 가이드 (허브)

- **카테고리**: #DBMS #MySQL #replication #허브
- **태그**: #replication #GTID #복제 #허브
- **작성일**: 2026-06-13
- **개정**: 2026-06-14 — 개별 문서 1:1 분리에 따라 **허브(목차) 문서로 전환**(중복 본문 제거)

## 1. 핵심 요약

> 이 문서는 MySQL **복제(Replication)** 주제의 개별 문서를 모은 **허브(목차)** 입니다.
> 상세 내용은 아래 개별 문서를 참조하세요. (원본 13개 → 개별 문서로 1:1 분리 완료)

---

## 2. 복제 방식 (설정)

- [[2026-06-13-01_(MySQL-Binary-Log-Position-복제)]] — Master Binary Log 파일/위치 기반 복제
- [[2026-06-13-02_(MySQL-GTID-기반-복제)]] — GTID 자동 위치(MASTER_AUTO_POSITION), 중복 실행 방지
- [[2026-06-13-03_(MySQL-Semi-Sync-복제)]] — Slave ACK 후 응답, 안전성>성능, timeout 관리
- [[2026-06-13-58_(MySQL-멀티-소스-복제-구성)]] — 다중 소스→1 replica, 채널별 CHANGE MASTER, semi-sync 불가

## 3. Slave 추가 방법

- [[2026-06-13-31_(MySQL-Slave-추가-CLONE-플러그인)]] — MySQL 8.0 CLONE 플러그인 물리 구성
- [[2026-06-13-32_(MySQL-Slave-추가-XtraBackup-GTID)]] — XtraBackup 물리백업 + GTID
- [[2026-06-13-33_(MySQL-Slave-추가-mysqldump)]] — 논리백업(mysqldump --master-data=2)

## 4. 운영 · 명령어 · 장애 처리

- [[2026-06-13-04_(MySQL-복제-명령어-모음)]] — CHANGE MASTER TO, SHOW SLAVE STATUS, GTID 관리
- [[2026-06-13-06_(MySQL-Replication-에러-처리-및-복구)]] — GTID/Position skip, 복제 재구성
- [[2026-06-13-34_(MySQL-Replication-재설정)]] — 복제 깨짐 시 GTID 전체 리셋 후 재동기화 10단계
- [[2026-06-13-59_(MySQL-Slave-gtid_executed-Master와-일치시키기)]] — RESET MASTER→gtid_purged 수동 동기화

## 5. 연관 개념

- [[2026-06-13-08_(MySQL-HA-Failover-테스트-전략)]]
- [[2026-06-13-20_(MySQL-Binary-Log-활성화-비활성화)]]
