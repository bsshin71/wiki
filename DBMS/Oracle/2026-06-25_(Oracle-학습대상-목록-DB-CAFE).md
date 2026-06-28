# Oracle 학습 대상 목록 (DB CAFE)

- **카테고리**: #DBMS #Oracle
- **태그**: #Oracle #학습로드맵 #개요 #DBA
- **작성일**: 2026-06-25
- **is_public**: true
- **참조 원본**: [[2026-06-25-ORACLE - 학습대상 목록]]

## 목차
- [[#1. 핵심 요약]]
- [[#1. 오라클 환경 구성]]
- [[#2. 오라클 관리]]
- [[#3. 오라클 성능 튜닝]]
- [[#4. 오라클 백업 복구]]
- [[#5. 오라클 TOOLS]]
- [[#연관 개념]]

## 1. 핵심 요약

DB CAFE Oracle 위키(출처: dbcafe.co.kr) 기반 학습 대상 목록. 환경구성·관리·성능튜닝·백업복구·TOOLS 5개 영역의 세부 Oracle DBA 학습 항목 목차. 개인 Oracle 학습 로드맵 및 미학습 항목 추적 용도로 활용.

---

> 출처: DB CAFE 위키 — ORACLE 페이지 「오라클 환경 구성」 영역

---

## 1. 오라클 환경 구성

### 1.1 오라클 데이터베이스 아키텍처

- 오라클 데이터베이스 구조
- 메모리 구조
- 프로세스 구조
- 논리적 / 물리적 저장영역 구조

### 1.2 오라클 데이터베이스 설치 / 환경 구성

- 오라클 데이터베이스 관리 도구
- 오라클 인벤토리(Inventory)
- 환경 변수
- 시스템 사전 요구 사항 검사
- Oracle 소프트웨어 설치

### 1.3 오라클 데이터베이스 생성

- DBCA
- DBCA 설정 옵션
- DBCA로 데이터베이스 생성

### 1.4 오라클 인스턴스 관리

- 초기화 파라미터
- PFILE 과 SPFILE의 특징
- 데이터베이스 시작
- 정상 종료 후 데이터베이스 시작 순서
- 데이터베이스 시작 명령어
- 데이터베이스 종료
- 데이터베이스 종료 모드
- 오라클 로그 관리
    - ADR
    - Alert log 파일
- Dynamic Performance 뷰
- Data Dictionary 뷰

### 1.5 오라클 네트워크 환경 구성

- Oracle Net 서비스
- Oracle Net 리스너
- 데이터베이스 서비스
- 리스너 로그 및 트레이스
- 이름 지정 방법
- Oracle Net 서비스 이름
- TNSPING 유틸리티
- 서버 프로세스 구조
- 데이터베이스 링크

---

## 2. 오라클 관리

### 2.1 오라클 데이터베이스 저장 관리

- 데이터 블록 구조
- 필수 테이블스페이스
- 테이블스페이스 생성
- 테이블스페이스 상태 및 변경
- 테이블스페이스 삭제
- 테이블스페이스 정보 조회
- 기본 영구 테이블스페이스
- 테이블스페이스 크기 확장
- 데이터 파일 크기 재지정
- 템프 테이블스페이스
- 언두 테이블스페이스

### 2.2 오라클 사용자 생성 / 권한 관리

- SYS와 SYSTEM 사용자
- 사용자 생성
- 사용자 계정 잠금 및 해제
- 권한
- 권한 수여 및 철회
- 롤 생성 및 관리
- Secure Application Role
- 프로파일 관리

### 2.3 오라클 오브젝트(테이블, 인덱스, 제약조건) 관리

- 데이터 유형
- 테이블 삭제
- 테이블 TRUNCATE
- PCTFREE
- 인덱스
- B-Tree Index
- 비트맵 인덱스
- 인덱스의 DML 효과
- 제약조건
- NOT NULL
- UNIQUE
- PRIMARY KEY
- FOREIGN KEY
- CHECK
- 지연가능 제약조건
- 제약조건 상태
- 임시 테이블

### 2.4 오라클 데이터 및 동시성 관리

- MERGE 명령어(문장)
- PL/SQL 객체 관리
- PL/SQL 객체 유형
- Package Specification과 Package Body
- 수동 테이블 잠금
- SELECT FOR UPDATE
- DDL_LOCK_TIMEOUT 파라미터
- 잠금 충돌 줄이기
- 잠금 충돌 해결
- 로우레벨 잠금 문제 자동 해결
- 데드락

### 2.5 오라클 언두 와 리두 테이블스페이스 관리

- 언두 데이터
- 언두 유형
- 트랜잭션과 언두 데이터
- 언두 테이블스페이스
- ORA-30036 에러
- ORA-01555 에러
- UNDO_RETENTION 파라미터
- 언두 보존 보장(Retention Guarantee)
- 고정 크기 언두 테이블스페이스

### 2.6 오라클 데이터베이스 유지 관리

- 옵티마이저 통계
- 옵티마이저 통계 수집
- 옵티마이저 통계 수집 환경 설정
- STATISTICS_LEVEL 파라미터
- 자동 유지관리 작업
- 유지관리 윈도우
- 사전 구성된 유지관리 윈도우
- AWR
- 베이스라인
- ADDM
- Server Generated Alert
- Enterprise Manager Support Workbench
- Quick Packaging과 Custom Packaging

### 2.7 SQL

- ORACLE SQL Collection
- SQL INSERT Statement
- ORACLE NVL NVL2
- NVL NVL2 Function
- LNNVL Function
- ORACLE COALESCE
- SQL CONNECT BY
- ORACLE SQL ADVISOR
- SQL LATERAL Statement
- SQL REGULAR Expressions
- SQL TRANSLATE Function
- SQL TRUNCATE Statement Usage
- ORACLE XML Query
- Analysis Function
- ORACLE PROCEDURE
- ORACLE PROCEDURE Sample
- ORACLE PROCEDURE Extraction Query
- ORACLE PROCEDURE FUNCTION LIST

### 2.8 Oracle Tools

- EXPORT DP
- SPM
- ORACLE LOGMINER
- ORACLE Sqlplus

---

## 3. 오라클 성능 튜닝

### 3.1 파라미터 성능 튜닝

- 메모리 관리
- 자동 메모리 관리(AMM)
- 자동 공유 메모리 관리(ASMM)
- 수동 튜닝 파라미터
- 수동 메모리 관리
- 그래뉼(granule)
- 무효화(Invalid) PL/SQL 객체 문제 해결
- 사용할 수 없는(Unusable) 인덱스 문제 해결

### 3.2 SQL 성능 튜닝

- ORACLE XPLAN
- DBMS XPLAN
- Oracle Subquery Description Join Tuning
- NL Join
- JPPD (Join Predicate PushDown)
- NL Join Prefetch
- HASH Join
- ORACLE HINT List
- SQL MERGE
- Tuning point per JOIN
- ORACLE JOIN Order

---

## 4. 오라클 백업 복구

### 4.1 오라클 백업 / 복구 관리

- 데이터베이스 장애 범주
- 인스턴스 복구(Instance Recovery)
- 체크포인트
- FAST_START_MTTR_TARGET 파라미터
- LOG_CHECKPOINT_INTERVAL 파라미터
- 체크포인트 튜닝
- 복구를 위한 구성
- 아카이브 로그 파일
- FRA(Flash Recovery Area)
- Flash Recovery Area 공간 회복
- ARCHIVELOG 모드 활성화
- 컨트롤 파일 다중화
- 리두 로그 파일 다중화

### 4.2 데이터베이스 백업 수행

- 백업 방식
- 백업 모드
- 사용자 관리 백업(User Managed Backup)
- RMAN
- RMAN 백업 설정 구성
- 증분 백업
- 백업 스케줄링
- RMAN 백업 관리
- OSB

### 4.3 데이터베이스 복구 수행

- 노아카이브 로그 모드에서 복구
- 컨트롤 파일 복구
- DRA
- DRA가 필요한 상황
- DRA 수동 체크리스트
- Health Monitor

---

## 5. 오라클 TOOLS

### 5.1 오라클 데이터 이동

- SQL*Loader
- SQL*Loader 로드 방식
- External Table
- 오라클 데이터 펌프
- Data Pump 데이터 이동 방법
- 데이터 펌프 엑스포트 / 임포트 모드
- REMAP_SCHEMA
- NETWORK_LINK

---

## 연관 개념

- [[2026-06-05_(Oracle-19c-Rocky-Linux-설치-가이드)]] — Oracle 19c 설치 실습 가이드
- [[2026-06-05_(Oracle-DBCA-NETCA-가이드)]] — DBCA 인스턴스 생성 및 리스너 구성
- [[2026-06-05_(Oracle-리스너-등록)]] — Oracle Net 리스너 설정
- [[2026-06-05_(Oracle-유저-생성하기)]] — 사용자 계정 생성 절차
- [[2026-06-02_(Oracle-모니터링-관리-쿼리-모음)]] — ASH·v$sql·Lock·Dictionary 조회 쿼리
- [[2026-06-15-OT01_(Oracle-SQL-튜닝-실무-12단계-가이드)]] — SQL 성능 튜닝 실무 가이드
- [[2026-06-16_(Oracle-주요-힌트-레퍼런스)]] — Oracle SQL 힌트 분류 레퍼런스
