# MySQL Lock · Deadlock 모니터링 (허브)

- **카테고리**: #DBMS #MySQL #lock #모니터링 #허브
- **태그**: #lock #deadlock #모니터링 #허브
- **작성일**: 2026-06-13
- **개정**: 2026-06-14 — 개별 문서 1:1 분리에 따라 **허브(목차) 문서로 전환**(중복 본문 제거)

## 1. 핵심 요약

> 이 문서는 MySQL **Lock · Deadlock · 세션 경합** 주제의 개별 문서를 모은 **허브(목차)** 입니다.
> 상세 내용은 아래 개별 문서를 참조하세요. (원본 8개 → 개별 문서로 1:1 분리 완료)

---

## 2. 개념 · 분석

- [[2026-06-13-16_(MySQL-Lock-개념-및-분석)]] — Record/Gap/Next-Key/Table/MDL 락, LOCK_MODE 해석, data_locks 조회
- [[2026-06-13-23_(MySQL-Deadlock-분석-및-해결)]] — Deadlock 자동 감지·롤백, SHOW ENGINE INNODB STATUS 분석
- [[2026-06-13-24_(MySQL-InnoDB-Lock-모니터링)]] — InnoDB Lock 모니터링, 성능 스키마 활용

## 3. 세션 · 블로킹 조회

- [[2026-06-13-17_(MySQL-Meta-Lock-세션-조회)]] — MDL 홀더 찾기, DDL 블로킹 원인 분석
- [[2026-06-13-55_(MySQL-블로킹-세션-조회-누가-누구를-막는지)]] — data_lock_waits·sys.schema_table_lock_waits, KILL 조치
- [[2026-06-13-22_(MySQL-미커밋-세션-조회)]] — INNODB_TRX로 미커밋 트랜잭션 조회·종료

## 4. 장애 사례 분석

- [[2026-06-13-56_(MySQL-Update-지연-사례-디스크-커밋-지연)]] — 락 아닌 디스크 커밋 지연(waiting for handler commit, iostat %util)
- [[2026-06-13-57_(MySQL-Update-지연-사례-레코드락-commit-지연)]] — 레코드락 + 다중 서버 중복 실행 + 앱 commit 지연

## 5. 연관 개념

- [[2026-06-13-52_(MySQL-오래-수행중인-TX-찾기)]]
- [[2026-06-13-29_(MySQL-Undo-로그-관리)]]
