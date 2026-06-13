# MySQL Percona Server 설치 가이드 (허브)

- **카테고리**: #DBMS #MySQL #install #허브
- **태그**: #install #Percona #설치 #허브
- **작성일**: 2026-06-13
- **개정**: 2026-06-14 — 개별 문서 1:1 분리에 따라 **허브(목차) 문서로 전환**(중복 본문 제거)

## 1. 핵심 요약

> 이 문서는 MySQL **Percona Server 설치·구성** 주제의 개별 문서를 모은 **허브(목차)** 입니다.
> 상세 내용은 아래 개별 문서를 참조하세요. (원본 8개 → 개별 문서로 1:1 분리 완료)

---

## 2. 설치

- [[2026-06-13-14_(MySQL-Percona-설치-상세-가이드)]] — 사용자/디렉터리, 의존성 해결, RPM 설치 순서, 초기 보안
- [[2026-06-13-15_(MySQL-Percona-설치-요약-체크리스트)]] — 5단계 빠른 참고(~25분)
- [[2026-06-13-40_(Percona-MySQL-devel-환경-구성)]] — C 클라이언트 devel 패키지, Bug#92870

## 3. 경로 · 버전 · 패치

- [[2026-06-13-41_(Percona-MySQL-datadir-basedir-경로-변경)]] — 비표준 경로, mysqld_pre_systemd 수정
- [[2026-06-13-39_(Percona-MySQL-8.4-vs-8.0-비교)]] — 8.4 LTS, 용어 변경, ProxySQL·MHA 호환
- [[2026-06-13-42_(Percona-Server-8.0.40-패치과정)]] — rpm -Uvh 업그레이드, libatomic·telemetry 의존성

## 4. 운영 설정 · 튜닝

- [[2026-06-13-43_(Percona-MySQL-운영-표준-my.cnf)]] — PRD 표준 my.cnf 항목별 설정
- [[2026-06-13-63_(MySQL-성능-최적화-커널-파라미터)]] — Linux sysctl·limits.conf 서버 튜닝

## 5. 연관 개념

- [[2026-06-13-08_(MySQL-HA-Failover-테스트-전략)]]
- [[2026-06-13-37_(MySQL-XtraBackup-설치-및-백업-계정-권한)]]
