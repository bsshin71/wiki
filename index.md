# 🌐 개인 학습 지식 지도

## 📚 학습 카테고리

* 이미 존재하는 카테고리를 우선 사용합니다.
* 적당한 카테고리가 없으면 AI가 지식이 쌓이면 자동으로 카테고리를 분류해 줄 예정입니다.

### 최상위 카테고리 (wiki/ root 직속)

| 카테고리 | 경로 | 설명 |
|----------|------|------|
| **시스템** | 시스템/ | OS·인프라·도구 설정 |
| **프로그래밍** | 프로그래밍/ | 프로그래밍 언어·프레임워크 |
| **DBMS** | DBMS/ | DB·쿼리튜닝·DW |
| **ML(AI)** | ML(AI)/ | ML·AI·딥러닝 |
| **kafka** | kafka/ | Apache Kafka |
| **elasticsearch** | elasticsearch/ | Elasticsearch |
| **airflow** | airflow/ | Apache Airflow |
| **기타(ETC)** | 기타(ETC)/ | 위 카테고리에 속하지 않는 기타 문서 |

### 전체 카테고리

| 카테고리 | 경로 | 설명 |
|---------|------|------|
| 시스템 | 시스템/ | 시스템·인프라·도구 설정 관련 문서 |
| 시스템/Obsidian | 시스템/ | Obsidian·MCP·Claude 도구 설정 가이드 |
| 프로그래밍 | 프로그래밍/ | 프로그래밍 언어·도구 관련 문서 |
| DBMS | DBMS/ | DB 공통 |
| PostgreSQL | DBMS/PostgreSQL/ | PostgreSQL 관련 문서 |
| PostgreSQL/admin | DBMS/PostgreSQL/admin/ | PostgreSQL 운영·모니터링·관리 쿼리 문서 |
| MySQL | DBMS/MySQL/ | MySQL 관련 문서 |
| Oracle | DBMS/Oracle/ | Oracle 관련 문서 |
| Oracle/install | DBMS/Oracle/install/ | Oracle 설치·구성 관련 문서 |
| Altibase | DBMS/Altibase/ | Altibase 관련 문서 |
| 쿼리튜닝 | DBMS/쿼리튜닝/ | DB 종류 무관 쿼리튜닝 (최우선 분류) |
| DW | DBMS/DW/ | 데이터웨어하우스 / 분석 관련 문서 |
| (공통) | DBMS/(공통)/ | DB 공통 개념 문서 |
| ML(AI) | ML(AI)/ | ML·AI·딥러닝 관련 문서 (독립 최상위) |
| kafka | kafka/ | Apache Kafka 관련 문서 (독립 최상위) |
| elasticsearch | elasticsearch/ | Elasticsearch 관련 문서 (독립 최상위) |
| airflow | airflow/ | Apache Airflow 관련 문서 (독립 최상위) |
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
- [[2026-06-02_(PostgreSQL-모니터링-쿼리-모음)]] : pg_stat_statements·pg_stat_activity·Block 관계·Cache Hit 비율 조회 `#PostgreSQL #admin #모니터링` `DBMS/PostgreSQL/admin/`
- [[2026-06-02_(PostgreSQL-객체-용량-조회-쿼리-모음)]] : DB 용량·Tablespace·Index·파티션·제약조건 조회 쿼리 모음 `#PostgreSQL #admin` `DBMS/PostgreSQL/admin/`
- [[2026-06-02_(PostgreSQL-운영-유틸리티-모음)]] : psql 명령어·파라미터 조회/변경·Auto Vacuum·테스트 데이터 생성 `#PostgreSQL #admin #psql` `DBMS/PostgreSQL/admin/`
- [[2026-06-02_(PostgreSQL-EXPLAIN-실행계획-가이드)]] : EXPLAIN 옵션 비교표·실행계획 노드·BUFFERS 해석·pg_hint_plan 힌트 `#PostgreSQL #쿼리튜닝 #EXPLAIN` `DBMS/쿼리튜닝/`
- [[2026-06-02_(PostgreSQL-pgBackRest-백업-가이드)]] : Stanza 생성→전체 백업→증분 백업→crontab 자동화, Tablespace 주의사항 `#PostgreSQL #backup` `DBMS/PostgreSQL/`
- [[2026-06-02_(PostgreSQL-pgvector-설치)]] : pgvector 확장 설치·벡터 테이블 생성·유사도 검색·IVFFlat/HNSW 인덱스 `#PostgreSQL #AI-RAG #pgvector` `DBMS/PostgreSQL/`
- [[2026-06-02_(PostgreSQL-MinTool4PG-DBA-도구)]] : Windows Terminal 환경용 PostgreSQL DBA 스크립트 도구 설치 및 pgpass.conf 설정 `#PostgreSQL #DBA #Windows` `DBMS/PostgreSQL/`
- [[2026-06-02_(MySQL-데이터이관-Shell-스크립트)]] : mysqldump 기반 테이블 단위 INSERT/REPLACE 이관 Shell 스크립트 `#MySQL #migration` `DBMS/MySQL/`
- [[2026-06-12_(MySQL-Lock-Deadlock-모니터링)]] : Lock 개념·종류·모니터링 쿼리·장애 사례(Update 지연이 Lock이 아닌 경우) `#MySQL #lock #모니터링` `DBMS/MySQL/`
- [[2026-06-12_(MySQL-Replication-가이드)]] : Binary Log Position / GTID 기반 복제 설정, Slave 추가(XtraBackup), Semi-Sync, 상황별 조치 `#MySQL #replication #GTID` `DBMS/MySQL/`
- [[2026-06-12_(MySQL-XtraBackup-백업복구-가이드)]] : Full/증분/시점 복구, innodb_force_recovery 긴급 장애 복구 `#MySQL #backup #XtraBackup` `DBMS/MySQL/`
- [[2026-06-12_(MySQL-Percona-설치-가이드)]] : RHEL/Rocky에 Percona Server RPM 설치, 운영용 my.cnf 전체 예시 `#MySQL #Percona #install` `DBMS/MySQL/`
- [[2026-06-12_(MySQL-Performance-Schema-활용)]] : sys 스키마 활용 SQL 성능 분석·인덱스 분석·메모리 분석·InnoDB 튜닝 `#MySQL #performance_schema #성능분석` `DBMS/쿼리튜닝/`
- [[2026-06-12_(MySQL-관리자-쿼리-모음)]] : User 생성·권한, 세션/TX 모니터링, General Log, Audit `#MySQL #admin #모니터링` `DBMS/MySQL/`
- [[2026-06-12_(MySQL-InnoDB-구조-설정)]] : Redo Log·Undo·Change Buffer·파티션·auto_increment·timezone·charset `#MySQL #InnoDB #구조` `DBMS/MySQL/`
- [[2026-06-02_(Linux-고정-IP-설정)]] : CentOS/RHEL ifcfg 파일로 고정 IP 설정, nmcli 강제 적용, DHCP 덮어쓰기 방지 `#Linux #네트워크 #IP고정` `시스템/`

---

## 📥 원본 처리 현황

> raw/ 원본 → wiki 문서 처리 추적. **여기 등록된 원본 = 처리 완료.**
> `/ingest` 시 raw/ 스캔 결과(attachments·pdf2md 제외)에서 이 목록에 **없는 파일 = 미처리**.
> ⛔ 원본은 이동·삭제하지 않고 raw/ 최초 위치에 그대로 둡니다.

### raw/ (루트)
- `2026-06-01-DbToys4Pg설치법.txt` → [[2026-06-02_(PostgreSQL-MinTool4PG-DBA-도구)]]
- `2026-06-02_spool로 export하는쿼리.md` → [[2026-06-02_(Oracle-spool-데이터-Export)]]
- `2026-06-05_Claudian-GitHub-MCP-설정가이드.md` → [[2026-06-05_(Claudian-GitHub-MCP-설정가이드)]]
- `2026-06-05_Claudian-Smart-Connections-MCP-설정가이드.md` → [[2026-06-05_(Claudian-Smart-Connections-MCP-설정가이드)]]
- `2026-06-12_LLMWiki-시스템-전체-가이드.md` → [[2026-06-12_(LLMWiki-시스템-전체-가이드)]]
- `2026-06-12_Obsidian Git 플러그인 연동가이드.md` → [[2026-06-12_(Obsidian-Git-플러그인-연동가이드)]]
- `block 관계 조회 쿼리.md` → [[2026-06-02_(PostgreSQL-모니터링-쿼리-모음)]]

### raw/pdf/
- `Advanced_SQL_PG_20250325_v0.9.pdf` → [[2026-06-08_(Advanced-SQL-PostgreSQL)]]

### raw/clippings/ (PG·Oracle·기타, 2026-06-02 / 2026-06-05)
- `2026-06-02-ASH(Active Session History)조회쿼리.md` → [[2026-06-02_(Oracle-모니터링-관리-쿼리-모음)]]
- `2026-06-02-v$sql수행통계정보조회쿼리.md` → [[2026-06-02_(Oracle-모니터링-관리-쿼리-모음)]]
- `2026-06-02-lock 모니터링 쿼리.md` → [[2026-06-02_(Oracle-모니터링-관리-쿼리-모음)]]
- `2026-06-02-dictionary table검색쿼리.md` → [[2026-06-02_(Oracle-모니터링-관리-쿼리-모음)]]
- `2026-06-02-snapshot too old 줄이기.md` → [[2026-06-02_(Oracle-쿼리튜닝-트러블슈팅)]]
- `2026-06-02-함수 부하가 병목인 쿼리 튜팅.md` → [[2026-06-02_(Oracle-쿼리튜닝-트러블슈팅)]]
- `2026-06-02-postresql rpm 설치과정.md` → [[2026-06-02_(PostgreSQL-RPM-설치-가이드)]]
- `2026-06-02-postresql설치(ver 18).md` → [[2026-06-02_(PostgreSQL-RPM-설치-가이드)]]
- `2026-06-02-Rocky9+PG설치 (rpm).md` → [[2026-06-02_(PostgreSQL-RPM-설치-가이드)]]
- `2026-06-02-PG explain 옵션.md` → [[2026-06-02_(PostgreSQL-EXPLAIN-실행계획-가이드)]]
- `2026-06-02-pg_stat_statements 활용.md` → [[2026-06-02_(PostgreSQL-모니터링-쿼리-모음)]]
- `2026-06-02-실행중인 쿼리 확인.md` → [[2026-06-02_(PostgreSQL-모니터링-쿼리-모음)]]
- `2026-06-02-database 별 cache hit  miss 비율 조회.md` → [[2026-06-02_(PostgreSQL-모니터링-쿼리-모음)]]
- `2026-06-02-PostgreSQL 통계 정보관련 view 테이블 종류.md` → [[2026-06-02_(PostgreSQL-모니터링-쿼리-모음)]]
- `2026-06-02-database size 조회쿼리.md` → [[2026-06-02_(PostgreSQL-객체-용량-조회-쿼리-모음)]]
- `2026-06-02-PostgreSQL 데이터베이스 용량 확인을 위한 SQL 쿼리.md` → [[2026-06-02_(PostgreSQL-객체-용량-조회-쿼리-모음)]]
- `2026-06-02-tablespace 정보 조회 쿼리.md` → [[2026-06-02_(PostgreSQL-객체-용량-조회-쿼리-모음)]]
- `2026-06-02-index와 table 이 사용하는 tablespace 조회쿼리.md` → [[2026-06-02_(PostgreSQL-객체-용량-조회-쿼리-모음)]]
- `2026-06-02-tabespace 를 사용하는 객체 조회.md` → [[2026-06-02_(PostgreSQL-객체-용량-조회-쿼리-모음)]]
- `2026-06-02-현재 데이터베이스의 인덱스 상세 정보 조회를 위한 SQL 쿼리.md` → [[2026-06-02_(PostgreSQL-객체-용량-조회-쿼리-모음)]]
- `2026-06-02-현재데이터베이스의 index 정보정보 조회.md` → [[2026-06-02_(PostgreSQL-객체-용량-조회-쿼리-모음)]]
- `2026-06-02-현재 데이터베이스의 모든  제약조건 확인.md` → [[2026-06-02_(PostgreSQL-객체-용량-조회-쿼리-모음)]]
- `2026-06-02-파티션 테이블 관계 조회 쿼리.md` → [[2026-06-02_(PostgreSQL-객체-용량-조회-쿼리-모음)]]
- `2026-06-02-파티션 테이블의 부모테이블과 자식테이블 조회.md` → [[2026-06-02_(PostgreSQL-객체-용량-조회-쿼리-모음)]]
- `2026-06-02-psql  주요 명령어.md` → [[2026-06-02_(PostgreSQL-운영-유틸리티-모음)]]
- `2026-06-02-시스템설정(Parameter) 조회 및 변경.md` → [[2026-06-02_(PostgreSQL-운영-유틸리티-모음)]]
- `2026-06-02-테이블에 설정된 auto vacuum 설정 확인.md` → [[2026-06-02_(PostgreSQL-운영-유틸리티-모음)]]
- `2026-06-02-테이블에 대량의 랜덤 데이터를 삽입하는 쿼리.md` → [[2026-06-02_(PostgreSQL-운영-유틸리티-모음)]]
- `2026-06-02-날짜 생성 및 판매 데이터 삽입을 위한 PLpgSQL 스크립트.md` → [[2026-06-02_(PostgreSQL-운영-유틸리티-모음)]]
- `2026-06-02-pgBackRest 초기화 및 전체 백업 수행 절차.md` → [[2026-06-02_(PostgreSQL-pgBackRest-백업-가이드)]]
- `2026-06-02-pg_vector 설치.md` → [[2026-06-02_(PostgreSQL-pgvector-설치)]]
- `2026-06-02-mysql데이터이관shellscript.md` → [[2026-06-02_(MySQL-데이터이관-Shell-스크립트)]]
- `2026-06-02-Linux 고정 IP 설정방법 - MyWiki.md` → [[2026-06-02_(Linux-고정-IP-설정)]]
- `2026-06-05-oracle 유저 생성하기.md` → [[2026-06-05_(Oracle-유저-생성하기)]]
- `2026-06-05-Rocky Linux 9.7 환경에 Oracle 19.3.0.0 설치과정.md` → [[2026-06-05_(Oracle-19c-Rocky-Linux-설치-가이드)]]
- `2026-06-05-listener 등록.md` → [[2026-06-05_(Oracle-리스너-등록)]]
- `2026-06-05-DBCA 와 NETCA 실행방법.md` → [[2026-06-05_(Oracle-DBCA-NETCA-가이드)]]

### raw/clippings/mysql/ (MySQL, 2026-06-12 · 71건)

**→ [[2026-06-12_(MySQL-Lock-Deadlock-모니터링)]]**
- `2026-06-12-mysql lock - BigDataTeam.md`, `2026-06-12-mysql deadlock - BigDataTeam.md`, `2026-06-12-Lock 개념 - BigDataTeam.md`, `2026-06-12-누가누구를 막고 있는지 보기 - BigDataTeam.md`, `2026-06-12-meta lock 을 잡고 있는 세션찾기 - BigDataTeam.md`, `2026-06-12-단순 Update문 지연사례 분석 - BigDataTeam.md`, `2026-06-12-Update 문 지연2 - BigDataTeam.md`, `2026-06-12-mysql commit 안하고 있는 세션찾기 - BigDataTeam.md`

**→ [[2026-06-12_(MySQL-Replication-가이드)]]**
- `2026-06-12-01. binary log file position based replication - BigDataTeam.md`, `2026-06-12-02. replication with global transaction identifiers - BigDataTeam.md`, `2026-06-12-03. semi sync replication - BigDataTeam.md`, `2026-06-12-05. replcation command 정리 - BigDataTeam.md`, `2026-06-12-07. replication 상황별 조치 - BigDataTeam.md`, `2026-06-12-semi replication - BigDataTeam.md`, `2026-06-12-replication 재설정 방법 - BigDataTeam.md`, `2026-06-12-slave 추가방법(cone statement) - BigDataTeam.md`, `2026-06-12-slave 추가방법(gtid based) - BigDataTeam.md`, `2026-06-12-slave 추가방법(mysqldump) - BigDataTeam.md`, `2026-06-12-slave 의 gtid_executed 를 master와 일치시키기 - BigDataTeam.md`, `2026-06-12-멀티 소스 복제 구성 - BigDataTeam.md`, `2026-06-12-BOS DB HA(failover) 테스트 - BigDataTeam.md`

**→ [[2026-06-12_(MySQL-XtraBackup-백업복구-가이드)]]**
- `2026-06-12-Xtrabackup Install setting - BigDataTeam.md`, `2026-06-12-XtraBackup의 동작원리 - BigDataTeam.md`, `2026-06-12-Xtrabackup 백업 요소 - BigDataTeam.md`, `2026-06-12-Full 백업 및 복구 - BigDataTeam.md`, `2026-06-12-증분 백업 - BigDataTeam.md`, `2026-06-12-증분 백업 복구 테스트 - BigDataTeam.md`, `2026-06-12-증분백업script(V8) - BigDataTeam.md`, `2026-06-12-full백업script ( V8 ) - BigDataTeam.md`, `2026-06-12-full백업script( v2.4 구버전용) - BigDataTeam.md`, `2026-06-12-백업 복구 시나리오 - BigDataTeam.md`, `2026-06-12-백업 복구 테스트 - BigDataTeam.md`, `2026-06-12-시점 복구 - BigDataTeam.md`, `2026-06-12-시점 복구 테스트 - BigDataTeam.md`, `2026-06-12-긴급장애 복구 - BigDataTeam.md`

**→ [[2026-06-12_(MySQL-Percona-설치-가이드)]]**
- `2026-06-12-install percona by rpm on redhat - BigDataTeam.md`, `2026-06-12-install percona by rpm on redhat(설치요약본) - BigDataTeam.md`, `2026-06-12-percona mysql devel 환경만들기 - BigDataTeam.md`, `2026-06-12-Percona MySQL 경로 수정 - BigDataTeam.md`, `2026-06-12-percona server 8.0.40 패치과정 - BigDataTeam.md`, `2026-06-12-percona mysql 8.4  vs percona mysql 8.0 - BigDataTeam.md`, `2026-06-12-prd db용 my.cnf - BigDataTeam.md`, `2026-06-12-성능 최적화를 위한 주요 커널파라미터 - BigDataTeam.md`

**→ [[2026-06-12_(MySQL-Performance-Schema-활용)]]**
- `2026-06-12-MySQL Performance_schema 활용 - BigDataTeam.md`, `2026-06-12-Performance 스키마를 이용한 프로파일링 - BigDataTeam.md`, `2026-06-12-events_statements_summary_by_digest를 이용하여 SQL성능 분석 - BigDataTeam.md`, `2026-06-12-MySQL 성능 튜닝 - BigDataTeam.md`, `2026-06-12-mysql optimization Index - BigDataTeam.md`, `2026-06-12-fast index creation - BigDataTeam.md`, `2026-06-12-데이터 Load 성능 올리기 - BigDataTeam.md`, `2026-06-12-mymon shell script for monitoring MySQL.md`

**→ [[2026-06-12_(MySQL-관리자-쿼리-모음)]]**
- `2026-06-12-mysql user 생성 - BigDataTeam.md`, `2026-06-12-mysql 권한 관리 - BigDataTeam.md`, `2026-06-12-sys schema 접근 권한 부여 - BigDataTeam.md`, `2026-06-12-MySQL Audit - BigDataTeam.md`, `2026-06-12-특정유저의 접속차단 - BigDataTeam.md`, `2026-06-12-03. MySQL admin 쿼리 - BigDataTeam.md`, `2026-06-12-기 실행된 쿼리내역 확인 - BigDataTeam.md`, `2026-06-12-오래 수행중인 TX 찾기 - BigDataTeam.md`, `2026-06-12-에러를 일으키는 client 프로세스 찾기 - BigDataTeam.md`, `2026-06-12-쿼리 로깅하기 - BigDataTeam.md`

**→ [[2026-06-12_(MySQL-InnoDB-구조-설정)]]**
- `2026-06-12-리두로그 및 로그버퍼 - BigDataTeam.md`, `2026-06-12-리두로그 아카이빙 - BigDataTeam.md`, `2026-06-12-리두로그 활성화 및 비활성화 - BigDataTeam.md`, `2026-06-12-체인지버퍼 - BigDataTeam.md`, `2026-06-12-mysql undo 관리 - BigDataTeam.md`, `2026-06-12-mysql partition 관리 - BigDataTeam.md`, `2026-06-12-auto_increment 의 사용시 주의점 - BigDataTeam.md`, `2026-06-12-MySQL timezone 설정 - BigDataTeam.md`, `2026-06-12-mysql character set 설정 - BigDataTeam.md`, `2026-06-12-MySQL bin log 활성비활성 작업 - BigDataTeam.md`
