# PostgreSQL 개요 (허브)

- **카테고리**: #DBMS #PostgreSQL #허브
- **태그**: #PostgreSQL #허브 #목차 #overview
- **작성일**: 2026-06-14

## 1. 핵심 요약

> 이 문서는 **PostgreSQL 카테고리의 탐색 진입점(허브)** 입니다. 문서를 `install`·`admin`·`monitoring`·`replication`·`maintenance` sub 폴더 + 기타로 분류해 목차로 제공합니다.

---

## 2. 📦 install/ — 설치 · 구조 · 구동

- [[2026-06-14-PG01_(PostgreSQL-소스-컴파일-설치)]] · [[2026-06-02_(PostgreSQL-RPM-설치-가이드)]]
- [[2026-06-14-PG02_(PostgreSQL-아키텍처-및-특징)]] — MVCC·MySQL 비교
- [[2026-06-14-PG03_(PostgreSQL-구동-및-종료-pg_ctl)]]

## 3. 🔧 admin/ — DB·Role·Schema·권한·객체 관리

- [[2026-06-14-PG04_(PostgreSQL-데이터베이스-관리)]] · [[2026-06-14-PG05_(PostgreSQL-사용자-Role-관리)]]
- [[2026-06-14-PG06_(PostgreSQL-스키마-관리-search_path)]] · [[2026-06-14-PG07_(PostgreSQL-권한-관리-GRANT)]]
- [[2026-06-14-PG09_(PostgreSQL-테이블스페이스-관리)]] · [[2026-06-14-PG10_(PostgreSQL-시퀀스-관리)]]
- [[2026-06-14-PG24_(PostgreSQL-psql-메타명령어-레퍼런스)]] · [[2026-06-14-PG25_(PostgreSQL-Access-Privileges-표기-해석)]]
- [[2026-06-02_(PostgreSQL-운영-유틸리티-모음)]] · [[2026-06-02_(PostgreSQL-객체-용량-조회-쿼리-모음)]]

## 4. 📊 monitoring/ — 모니터링 · 세션 · 문제 진단

- [[2026-06-14-PG08_(PostgreSQL-세션-관리-연결제어)]] · [[2026-06-14-PG22_(PostgreSQL-too-many-connections-오류)]]
- [[2026-06-14-PG19_(PostgreSQL-모니터링-시스템뷰-pg_stat_activity)]] · [[2026-06-14-PG20_(PostgreSQL-상황별-모니터링-쿼리)]]
- [[2026-06-14-PG21_(PostgreSQL-문제상황-Index-Bloating-Deadlock)]]
- [[2026-06-02_(PostgreSQL-모니터링-쿼리-모음)]]

## 5. 🔁 replication/ — 복제 · HA

- [[2026-06-14-PG15_(PostgreSQL-복제-방식-물리-논리)]] · [[2026-06-14-PG16_(PostgreSQL-복제-구성-실습-Streaming-Logical)]]
- [[2026-06-14-PG18_(PostgreSQL-Patroni-HA-구성)]]

## 6. 🧹 maintenance/ — 파티션 · pg_cron · Vacuum · 백업

- [[2026-06-14-PG11_(PostgreSQL-pg_partman-파티션-자동관리)]] · [[2026-06-14-PG13_(PostgreSQL-Online-파티션-테이블-재구성)]]
- [[2026-06-14-PG12_(PostgreSQL-pg_cron-batch-스케줄링)]]
- [[2026-06-14-PG17_(PostgreSQL-AutoVacuum-설정-및-튜닝)]] · [[2026-06-14-PG27_(PostgreSQL-Vacuum-개념-MVCC-XID-Wraparound-Freeze)]]
- [[2026-06-02_(PostgreSQL-pgBackRest-백업-가이드)]]

## 7. 기타 · 쿼리튜닝

- [[2026-06-14-PG14_(PostgreSQL-pg_dump-테이블-DDL-추출)]] · [[2026-06-14-PG23_(PostgreSQL-mysql_fdw-MySQL-연동)]] · [[2026-06-14-PG26_(PostgreSQL-Citus-분산-샤딩)]]
- [[2026-06-02_(PostgreSQL-pgvector-설치)]] · [[2026-06-02_(PostgreSQL-MinTool4PG-DBA-도구)]]
- [[2026-06-02_(PostgreSQL-EXPLAIN-실행계획-가이드)]] · [[2026-06-08_(Advanced-SQL-PostgreSQL)]]

## 8. 연관 개념 (타 카테고리 연결)

- [[2026-06-14-PG23_(PostgreSQL-mysql_fdw-MySQL-연동)]] ↔ [[2026-06-02_(MySQL-데이터이관-Shell-스크립트)]]
- [[2026-06-14-Kafka-개요-허브]] · [[2026-06-14-Elasticsearch-개요-허브]]
