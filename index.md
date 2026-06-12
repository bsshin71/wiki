# 🌐 개인 학습 지식 지도

## 📚 학습 카테고리

* 이미 존재하는 카테고리를 우선 사용합니다.
* 적당한 카테고리가 없으면 AI가 지식이 쌓이면 자동으로 카테고리를 분류해 줄 예정입니다.

### 주요 카테고리

| # | 카테고리 | 경로 | 설명 |
|---|----------|------|------|
| 1 | **시스템** | 시스템/ | OS·인프라·도구 설정 |
| 2 | **프로그래밍** | 프로그래밍/ | 프로그래밍 언어·프레임워크 |
| 3 | **DBMS** | DBMS/ | DB·쿼리튜닝·DW |
| 4 | **기타(ETC)** | 기타(ETC)/ | 위 카테고리에 속하지 않는 기타 문서 |

### 전체 카테고리

| 카테고리 | 경로 | 설명 |
|---------|------|------|
| 시스템 | 시스템/ | 시스템·인프라·도구 설정 관련 문서 |
| 시스템/Obsidian | 시스템/ | Obsidian·MCP·Claude 도구 설정 가이드 |
| 프로그래밍 | 프로그래밍/ | 프로그래밍 언어·도구 관련 문서 |
| DBMS | DBMS/ | DB 공통 |
| PostgreSQL | DBMS/PostgreSQL/ | PostgreSQL 관련 문서 |
| MySQL | DBMS/MySQL/ | MySQL 관련 문서 |
| Oracle | DBMS/Oracle/ | Oracle 관련 문서 |
| Oracle/install | DBMS/Oracle/install/ | Oracle 설치·구성 관련 문서 |
| Altibase | DBMS/Altibase/ | Altibase 관련 문서 |
| 쿼리튜닝 | DBMS/쿼리튜닝/ | DB 종류 무관 쿼리튜닝 (최우선 분류) |
| DW | DBMS/DW/ | 데이터웨어하우스 / 분석 관련 문서 |
| (공통) | DBMS/(공통)/ | DB 공통 개념 문서 |
| ML(AI) | ML(AI)/ | ML·AI·딥러닝 관련 문서 (독립 최상위) |
| 기타(ETC) | 기타(ETC)/ | 위 카테고리에 속하지 않는 기타 문서 |

---

## 📄 전체 문서 인덱스

> 형식: `[[문서명]]` : 설명 `#태그1 #태그2` `경로`

- [[2026-06-08_(Advanced-SQL-PostgreSQL)]] : Oracle→PG 전환 관점의 Advanced SQL 실습 가이드 — INDEX·분석함수·ROLLUP·계층질의·이행 체크리스트 `#PostgreSQL #AdvancedSQL #Oracle이행` `DBMS/PostgreSQL/`
- [[2026-06-05_(Oracle-유저-생성하기)]] : Oracle 19c 사용자 계정 생성 4단계 절차 및 ORA-65096 에러 대처 `#Oracle #install` `DBMS/Oracle/install/`
- [[2026-06-05_(Oracle-19c-Rocky-Linux-설치-가이드)]] : Rocky Linux 9.7에서 Oracle 19c 설치 시 GCC 11+ 충돌 해결 마스터 가이드 `#Oracle #install` `DBMS/Oracle/install/`
- [[2026-06-05_(Oracle-리스너-등록)]] : 동적 등록(자동)과 정적 등록(listener.ora 수동) 비교 및 사용 시나리오 `#Oracle #install` `DBMS/Oracle/install/`
- [[2026-06-05_(Oracle-DBCA-NETCA-가이드)]] : Oracle 19c DB 인스턴스 무인 생성(DBCA) 및 리스너 구성(NETCA) 마스터 가이드 `#Oracle #install` `DBMS/Oracle/install/`
- [[2026-06-05_(Claudian-Smart-Connections-MCP-설정가이드)]] : Obsidian+Claudian 환경에서 Smart Connections MCP 설정 — 벡터 검색으로 vault 탐색 토큰 절감, nvm 심볼릭 링크 함정 해결 `#obsidian #mcp` `시스템/`
- [[2026-06-05_(Claudian-GitHub-MCP-설정가이드)]] : Obsidian+Claudian 환경에서 GitHub MCP 설정 — vault 문서를 GitHub 레포에 자연어로 업로드, PAT 보안 주의사항 `#obsidian #mcp #github` `시스템/`
- [[2026-06-12_(LLMWiki-Graphify-통합-설정-로그)]] : graphify 지식 그래프 연동 설정 로그 — robocopy 왕복 방식으로 출력 경로 문제 해결, /ingest·/lint 토큰 절감 설계 `#graphify #설정로그` `시스템/`
- [[2026-06-12_(Obsidian-Git-플러그인-연동가이드)]] : Obsidian Git 플러그인으로 wiki/ 폴더만 GitHub Private 레포에 선택적 동기화 — Custom base path 설정·자동 백업 주기·첫 푸시 절차 `#git #github #backup` `시스템/`
- [[2026-06-12_(LLMWiki-시스템-전체-가이드)]] : LLM Wiki 전체 구조·프로세스·구축 과정·Obsidian 플러그인 통합 가이드 `#wiki #obsidian #graphify #mcp` `ML(AI)/`
- [[2026-06-02_(Oracle-모니터링-관리-쿼리-모음)]] : ASH·v$sql·Lock·Dictionary table 조회 쿼리 `#Oracle #admin #모니터링` `DBMS/Oracle/`
- [[2026-06-02_(Oracle-쿼리튜닝-트러블슈팅)]] : Snapshot too old 원인·해결, SELECT 절 사용자 함수 병목 튜닝 4가지 방법 `#Oracle #쿼리튜닝 #tibero` `DBMS/쿼리튜닝/`
- [[2026-06-02_(Oracle-spool-데이터-Export)]] : SQL*Plus spool을 이용한 테이블 데이터 Export 스크립트 `#Oracle #export #spool` `DBMS/Oracle/`
- [[2026-06-02_(PostgreSQL-RPM-설치-가이드)]] : RHEL/Rocky Linux에서 공식 RPM으로 PG16·PG18 설치 절차 및 데이터 경로 변경 `#PostgreSQL #install` `DBMS/PostgreSQL/`
- [[2026-06-02_(PostgreSQL-모니터링-쿼리-모음)]] : pg_stat_statements·pg_stat_activity·Block 관계·Cache Hit 비율 조회 `#PostgreSQL #admin #모니터링` `DBMS/PostgreSQL/`
- [[2026-06-02_(PostgreSQL-객체-용량-조회-쿼리-모음)]] : DB 용량·Tablespace·Index·파티션·제약조건 조회 쿼리 모음 `#PostgreSQL #admin` `DBMS/PostgreSQL/`
- [[2026-06-02_(PostgreSQL-운영-유틸리티-모음)]] : psql 명령어·파라미터 조회/변경·Auto Vacuum·테스트 데이터 생성 `#PostgreSQL #admin #psql` `DBMS/PostgreSQL/`
- [[2026-06-02_(PostgreSQL-EXPLAIN-실행계획-가이드)]] : EXPLAIN 옵션 비교표·실행계획 노드·BUFFERS 해석·pg_hint_plan 힌트 `#PostgreSQL #쿼리튜닝 #EXPLAIN` `DBMS/쿼리튜닝/`
- [[2026-06-02_(PostgreSQL-pgBackRest-백업-가이드)]] : Stanza 생성→전체 백업→증분 백업→crontab 자동화, Tablespace 주의사항 `#PostgreSQL #backup` `DBMS/PostgreSQL/`
- [[2026-06-02_(PostgreSQL-pgvector-설치)]] : pgvector 확장 설치·벡터 테이블 생성·유사도 검색·IVFFlat/HNSW 인덱스 `#PostgreSQL #AI-RAG #pgvector` `DBMS/PostgreSQL/`
- [[2026-06-02_(PostgreSQL-MinTool4PG-DBA-도구)]] : Windows Terminal 환경용 PostgreSQL DBA 스크립트 도구 설치 및 pgpass.conf 설정 `#PostgreSQL #DBA #Windows` `DBMS/PostgreSQL/`
- [[2026-06-02_(MySQL-데이터이관-Shell-스크립트)]] : mysqldump 기반 테이블 단위 INSERT/REPLACE 이관 Shell 스크립트 `#MySQL #migration` `DBMS/MySQL/`
- [[2026-06-02_(Linux-고정-IP-설정)]] : CentOS/RHEL ifcfg 파일로 고정 IP 설정, nmcli 강제 적용, DHCP 덮어쓰기 방지 `#Linux #네트워크 #IP고정` `시스템/`
