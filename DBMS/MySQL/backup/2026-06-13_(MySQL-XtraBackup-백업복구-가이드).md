# MySQL XtraBackup 백업복구 가이드 (허브)

- **카테고리**: #DBMS #MySQL #backup #허브
- **태그**: #backup #XtraBackup #복구 #허브
- **작성일**: 2026-06-13
- **개정**: 2026-06-14 — 개별 문서 1:1 분리에 따라 **허브(목차) 문서로 전환**(중복 본문 제거)

## 1. 핵심 요약

> 이 문서는 MySQL **백업·복구(XtraBackup/innobackupex)** 주제의 개별 문서를 모은 **허브(목차)** 입니다.
> 상세 내용은 아래 개별 문서를 참조하세요. (원본 14개 → 개별 문서로 1:1 분리 완료)

---

## 2. XtraBackup 원리 · 설치 · 옵션

- [[2026-06-13-35_(MySQL-XtraBackup-동작원리)]] — Hot Backup 원리, redo log 추적, LSN 일관성
- [[2026-06-13-36_(MySQL-XtraBackup-백업-요소-및-옵션)]] — 메타파일(checkpoints·binlog_info), prepare/copy-back/apply-log-only
- [[2026-06-13-37_(MySQL-XtraBackup-설치-및-백업-계정-권한)]] — percona-release 설치, 버전별 백업 계정 권한

## 3. Full / 증분 백업 · 스크립트

- [[2026-06-13-11_(MySQL-Full-Backup-복구-innobackupex)]] — innobackupex full backup/recovery
- [[2026-06-13-12_(MySQL-Percona-V8-Full-Backup-스크립트)]] — V8 스크립트(BACKUP_ADMIN, 병렬·스트림)
- [[2026-06-13-13_(MySQL-Percona-v2.4-Legacy-Backup-스크립트)]] — innobackupex 레거시(5.6/5.7)
- [[2026-06-13-38_(MySQL-XtraBackup-증분-백업-및-복구)]] — 증분 시나리오, --redo-only 규칙
- [[2026-06-13-64_(MySQL-증분-백업-자동화-스크립트-LSN기반)]] — LSN 자동 전달 스크립트, PXB-2406 버그

## 4. 복구 시나리오

- [[2026-06-13-44_(MySQL-시점-복구-PITR)]] — 전체 백업 + binlog 재적용, mysqlbinlog position
- [[2026-06-13-45_(MySQL-백업-복구-시나리오)]] — 단일테이블·온라인/아카이브 binlog·DROP 복구
- [[2026-06-13-46_(MySQL-긴급장애-복구-innodb_force_recovery)]] — force_recovery(1~6) 강제기동

## 5. 연관 개념

- [[2026-06-13-26_(MySQL-Redo-Log-및-로그버퍼-설정)]]
- [[2026-06-13-20_(MySQL-Binary-Log-활성화-비활성화)]]
