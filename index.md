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
- [[2026-06-13_(MySQL-Lock-Deadlock-모니터링)]] : 🗂️**허브** — Lock·Deadlock·세션경합·장애사례 개별문서 목차(16,17,23,24,55,56,57) `#MySQL #lock #허브` `DBMS/MySQL/lock/`
- [[2026-06-13_(MySQL-Replication-가이드)]] : 🗂️**허브** — 복제 방식·Slave 추가·운영/장애 개별문서 목차(01~04,06,31~34,58,59) `#MySQL #replication #허브` `DBMS/MySQL/replication/`
- [[2026-06-13_(MySQL-XtraBackup-백업복구-가이드)]] : 🗂️**허브** — XtraBackup 원리·Full/증분·복구 시나리오 개별문서 목차(11,35~38,44~46,64) `#MySQL #backup #허브` `DBMS/MySQL/backup/`
- [[2026-06-13_(MySQL-Percona-설치-가이드)]] : 🗂️**허브** — Percona 설치·경로·패치·운영설정 개별문서 목차(14,15,39~43,63) `#MySQL #install #허브` `DBMS/MySQL/installation/`
- [[2026-06-13_(MySQL-관리자-쿼리-모음)]] : 🗂️**허브** — admin 기초·계정/권한·세션진단 개별문서 목차(05,18,19,47~54) `#MySQL #admin #허브` `DBMS/MySQL/admin/`
- [[2026-06-13_(MySQL-Performance-Schema-활용)]] : PS 활성화·초기화, 접속/세션 모니터링, digest SQL분석, sys뷰, 인덱스·테이블 I/O, 프로파일링(stage 추적), 메모리분석, InnoDB튜닝, 오버헤드 — 구버전(2026-06-12) 통합 `#MySQL #performance_schema #sys #성능분석` `DBMS/쿼리튜닝/`
- [[2026-06-13-01_(MySQL-Binary-Log-Position-복제)]] : Master Binary Log 파일/위치 기반 복제. 초기 데이터 동기화(mysqldump/cold backup), Slave 설정, 검증. `#MySQL #replication` `DBMS/MySQL/replication/`
- [[2026-06-13-02_(MySQL-GTID-기반-복제)]] : GTID(Global Transaction ID)로 유일한 트랜잭션 식별. MASTER_AUTO_POSITION으로 자동 위치 결정, 중복 실행 방지, mysql.gtid_executed 테이블. `#MySQL #GTID #replication` `DBMS/MySQL/replication/`
- [[2026-06-13-03_(MySQL-Semi-Sync-복제)]] : Slave Relay Log 기록 ACK 후 Master 응답. 안전성 > 성능, Semi-Sync 플러그인 설정, timeout 관리, 모니터링. `#MySQL #replication` `DBMS/MySQL/replication/`
- [[2026-06-13-04_(MySQL-복제-명령어-모음)]] : Master/Slave 상태 확인, CHANGE MASTER TO, SHOW SLAVE STATUS, 에러 처리, GTID 관리 명령어 정리. `#MySQL #replication` `DBMS/MySQL/replication/`
- [[2026-06-13-05_(MySQL-Admin-쿼리-기초)]] : 버전 확인, DB/테이블/파티션 용량, 인덱스 분석(미사용), Primary Key 조회, Slow Query 로그, 연결 관리. `#MySQL #admin #모니터링` `DBMS/MySQL/admin/`
- [[2026-06-13-06_(MySQL-Replication-에러-처리-및-복구)]] : Slave 복제 중단 에러 원인 분석. GTID 건너뛰기(SET gtid_next), Position 기반 skip_counter, 전체 복제 재구성. 예방 체크리스트. `#MySQL #replication` `DBMS/MySQL/replication/`
- [[2026-06-13-07_(MySQL-auto_increment-주의점-및-최적화)]] : PK/UNIQUE 속성 필수, 증가값 감소 불가, InnoDB 재시작 시 MAX+1로 초기화. Rollback 후 채번값 건너뜀. Replication 불동기화. Failover 위험. `#MySQL #schema` `DBMS/MySQL/`
- [[2026-06-13-08_(MySQL-HA-Failover-테스트-전략)]] : ProxySQL·MHA·PMM 아키텍처. Client/Server 테스트. Simple Insert & 주문 부하 테스트. Master 강제 종료 & failover 검증. 체크리스트. `#MySQL #HA` `DBMS/MySQL/`
- [[2026-06-13-09_(MySQL-SQL성능-분석-Performance-Schema)]] : events_statements_summary_by_digest 활용. 느린 쿼리·정렬 최적화·인덱스 분석·에러 감지. sort_buffer_size, tmp_table_size 튜닝. `#MySQL #Performance` `DBMS/MySQL/`
- [[2026-06-13-10_(MySQL-Fast-Index-Creation-최적화)]] : expand_fast_index_creation 활성화. sql_log_bin=OFF. 버퍼풀 크기 조정. 550M 테이블: 5시간 → 4시간 (20% 개선). Secondary Index 30% 개선. `#MySQL #Indexing` `DBMS/MySQL/`
- [[2026-06-13-11_(MySQL-Full-Backup-복구-innobackupex)]] : innobackupex full backup/recovery. LSN 추적, InnoDB+non-InnoDB. --apply-log prepare, --copy-back restore. 에러 처리(권한, socket). `#MySQL #Backup` `DBMS/MySQL/backup/`
- [[2026-06-13-12_(MySQL-Percona-V8-Full-Backup-스크립트)]] : V8 사용자(BACKUP_ADMIN 권한), --compress-threads(v8.0.35+), 병렬백업, 스트림 전송, Archive Binlog 백업. Crontab 설정. `#MySQL #Backup` `DBMS/MySQL/backup/`
- [[2026-06-13-13_(MySQL-Percona-v2.4-Legacy-Backup-스크립트)]] : innobackupex 레거시, RELOAD 권한만 필요, --apply-log, MySQL 5.6/5.7 호환. MyISAM --no-lock 주의. `#MySQL #Backup` `DBMS/MySQL/backup/`
- [[2026-06-13-14_(MySQL-Percona-설치-상세-가이드)]] : 사용자/그룹 생성, 디렉터리 설정. compat-openssl10 의존성 해결. RPM 설치 순서. 커널 파라미터(ulimit). 초기 보안 설정. `#MySQL #Installation` `DBMS/MySQL/installation/`
- [[2026-06-13-15_(MySQL-Percona-설치-요약-체크리스트)]] : 설치 요약 (5단계, ~25분). 사전 준비 → 의존성 → RPM 설치 → 초기화 → 검증. 빠른 참고용 체크리스트. `#MySQL #Installation` `DBMS/MySQL/installation/`
- [[2026-06-13-16_(MySQL-Lock-개념-및-분석)]] : Record/Gap/Next-Key/Table/MDL 락. LOCK_MODE 해석(IX/IS/S/X/MDL_EXCLUSIVE). data_locks 조회. 원인 쿼리 판단. `#MySQL #Lock` `DBMS/MySQL/lock/`
- [[2026-06-13-17_(MySQL-Meta-Lock-세션-조회)]] : MDL 홀더 찾기. DDL 블로킹 원인 분석. ALTER/DROP 실패 시 세션 종료. `#MySQL #Lock` `DBMS/MySQL/lock/`
- [[2026-06-13-18_(MySQL-모니터링-Shell-스크립트)]] : mymon 스크립트. Threads, Questions, Slow_queries, Replication lag. 실시간 모니터링. `#MySQL #Monitoring` `DBMS/MySQL/`
- [[2026-06-13-19_(MySQL-Audit-로깅)]] : Percona Audit Plugin. 쿼리/로그인 기록(XML/JSON). GDPR/SOX 규정 준수. `#MySQL #Audit` `DBMS/MySQL/`
- [[2026-06-13-20_(MySQL-Binary-Log-활성화-비활성화)]] : sql_log_bin=OFF (SESSION). Master/Slave 주의. 선택적 로깅(binlog_do_db/binlog_ignore_db). `#MySQL #BinLog` `DBMS/MySQL/`
- [[2026-06-13-21_(MySQL-Character-Set-설정)]] : Character Set 불일치는 데이터 손상·복제 오류 초래. Server/Database/Table/Column 레벨 개별 설정 가능. utf8mb4 권장(emoji·다국어). `#MySQL #Schema` `DBMS/MySQL/`
- [[2026-06-13-22_(MySQL-미커밋-세션-조회)]] : COMMIT하지 않은 트랜잭션은 락 유지하여 쿼리 블로킹. INFORMATION_SCHEMA.INNODB_TRX로 조회·강제 종료. `#MySQL #Transaction` `DBMS/MySQL/admin/`
- [[2026-06-13-23_(MySQL-Deadlock-분석-및-해결)]] : Deadlock은 다중 트랜잭션이 서로 락 대기. InnoDB 자동 감지·롤백. SHOW ENGINE INNODB STATUS로 분석. `#MySQL #Lock` `DBMS/MySQL/lock/`
- [[2026-06-13-24_(MySQL-InnoDB-Lock-모니터링)]] : InnoDB Lock 실시간 모니터링, performance_schema.data_locks·성능 스키마 활용. `#MySQL #Lock #모니터링` `DBMS/MySQL/lock/`
- [[2026-06-13-25_(MySQL-Index-최적화)]] : 적절한 인덱스로 쿼리 성능 10배 향상. 선택도(Cardinality)·조건·정렬 순서로 복합 인덱스 설계. `#MySQL #Indexing` `DBMS/MySQL/`
- [[2026-06-13-26_(MySQL-Redo-Log-및-로그버퍼-설정)]] : innodb_flush_log_at_trx_commit(0/1/2) 내구성·성능 조절, log_file_size·log_buffer_size 파라미터. `#MySQL #InnoDB #redo_log` `DBMS/MySQL/innodb/`
- [[2026-06-13-27_(MySQL-Redo-Log-아카이빙)]] : MySQL 8.0 redo log 아카이빙. archive_dirs 설정, archive_start/stop, chmod 700 필수, 시작 세션 유지 필요. `#MySQL #InnoDB #redo_log` `DBMS/MySQL/innodb/`
- [[2026-06-13-28_(MySQL-Redo-Log-활성화-비활성화)]] : 대량 적재 시 redo log 비활성화로 시간 단축(8.0+). 비정상 종료 시 innodb_force_recovery=6 필요. `#MySQL #InnoDB #redo_log` `DBMS/MySQL/innodb/`
- [[2026-06-13-29_(MySQL-Undo-로그-관리)]] : undo 사용량·파일 조회, 테이블스페이스 추가/삭제(INACTIVE), 자동/수동 truncate 공간 회수. `#MySQL #InnoDB #undo` `DBMS/MySQL/innodb/`
- [[2026-06-13-30_(MySQL-Change-Buffer)]] : 세컨더리 인덱스 변경 버퍼링. innodb_change_buffering(all/inserts/...) 설정, ibuf0ibuf 메모리 조회. `#MySQL #InnoDB #change_buffer` `DBMS/MySQL/innodb/`
- [[2026-06-13-31_(MySQL-Slave-추가-CLONE-플러그인)]] : MySQL 8.0 CLONE 플러그인으로 Slave 물리 구성. Donor/Recipient 권한, CLONE INSTANCE, clone_progress, UUID 충돌 시 auto.cnf 삭제. `#MySQL #replication #CLONE` `DBMS/MySQL/replication/`
- [[2026-06-13-32_(MySQL-Slave-추가-XtraBackup-GTID)]] : XtraBackup 물리백업+GTID로 Slave 구성. xtrabackup_binlog_info의 GTID→gtid_purged, copy-back, AUTO_POSITION. `#MySQL #replication #XtraBackup #GTID` `DBMS/MySQL/replication/`
- [[2026-06-13-33_(MySQL-Slave-추가-mysqldump)]] : 논리백업(mysqldump --master-data=2)으로 Slave 구성. 덤프 내 gtid_purged 자동 설정, 소량 데이터 적합. `#MySQL #replication #mysqldump` `DBMS/MySQL/replication/`
- [[2026-06-13-34_(MySQL-Replication-재설정)]] : 복제 깨짐 시 Master/Slave GTID 전체 리셋 후 mysqldump 재동기화 10단계. RESET MASTER로 gtid_executed 비우기. `#MySQL #replication #재설정` `DBMS/MySQL/replication/`
- [[2026-06-13-35_(MySQL-XtraBackup-동작원리)]] : Hot Backup 원리. redo log 추적→별도 보관, prepare로 Roll Forward, LSN 일관성, MyISAM은 FTWRL. `#MySQL #backup #XtraBackup` `DBMS/MySQL/backup/`
- [[2026-06-13-36_(MySQL-XtraBackup-백업-요소-및-옵션)]] : 메타파일(checkpoints·binlog_info·info), 주요 옵션(prepare·copy-back·apply-log-only), 증분 복구 시 apply-log-only 필수. `#MySQL #backup #XtraBackup` `DBMS/MySQL/backup/`
- [[2026-06-13-37_(MySQL-XtraBackup-설치-및-백업-계정-권한)]] : percona-release 설치, 버전별(2.4/8.0/8.0.35+) 백업 계정 권한 차이, compress-threads 옵션명·native_password 주의. `#MySQL #backup #XtraBackup #설치` `DBMS/MySQL/backup/`
- [[2026-06-13-38_(MySQL-XtraBackup-증분-백업-및-복구)]] : 증분 백업 시나리오(LSN basedir), --redo-only 규칙(마지막 증분만 롤백), 복구 절차·오류대처·crontab 자동화. `#MySQL #backup #증분백업` `DBMS/MySQL/backup/`
- [[2026-06-13-39_(Percona-MySQL-8.4-vs-8.0-비교)]] : 8.4 LTS 특징, MASTER/SLAVE→SOURCE/REPLICA 용어 변경, ProxySQL·MHA 호환 문제로 8.0 유지 권고. `#MySQL #install #version84` `DBMS/MySQL/installation/`
- [[2026-06-13-40_(Percona-MySQL-devel-환경-구성)]] : C 클라이언트 devel/shared 패키지, Bug#92870(8.0.21+), mariadb 라이브러리 제거, libmysqlclient 컴파일 테스트. `#MySQL #install #devel` `DBMS/MySQL/installation/`
- [[2026-06-13-41_(Percona-MySQL-datadir-basedir-경로-변경)]] : datadir/basedir 비표준 경로 변경, my.cnf 수정·초기화, mysqld_pre_systemd 수정으로 systemd가 my.cnf 참조. `#MySQL #install #datadir` `DBMS/MySQL/installation/`
- [[2026-06-13-42_(Percona-Server-8.0.40-패치과정)]] : rpm -Uvh 업그레이드, libatomic·percona-telemetry-agent 의존성 해결, telemetry 비활성화. `#MySQL #install #patch` `DBMS/MySQL/installation/`
- [[2026-06-13-43_(Percona-MySQL-운영-표준-my.cnf)]] : PRD 표준 my.cnf 항목별 설정값(연결·InnoDB·복제·로그·보안), READ-COMMITTED·buffer_pool 8G·doublewrite OFF. `#MySQL #install #my.cnf` `DBMS/MySQL/installation/`
- [[2026-06-13-44_(MySQL-시점-복구-PITR)]] : 전체 백업+백업 이후 binlog 재적용. xtrabackup_binlog_info position, mysqlbinlog --start/stop-position, 복구 테스트 검증. `#MySQL #backup #PITR` `DBMS/MySQL/backup/`
- [[2026-06-13-45_(MySQL-백업-복구-시나리오)]] : 상황별 복구 4사례 — 단일테이블(DISCARD/IMPORT), 온라인/아카이브 binlog 시점복구, 실수 DROP 복구(drop 구문 주석), qpress 압축복구 검증. `#MySQL #backup #복구` `DBMS/MySQL/backup/`
- [[2026-06-13-46_(MySQL-긴급장애-복구-innodb_force_recovery)]] : InnoDB 자동복구 실패 시 force_recovery(1~6) 강제기동 후 mysqldump 재구축. 값별 모드·손상유형별 적용순서. `#MySQL #backup #장애복구` `DBMS/MySQL/backup/`
- [[2026-06-13-47_(MySQL-User-계정-생성-옵션)]] : CREATE USER 옵션 — IDENTIFIED WITH, PASSWORD EXPIRE/HISTORY/REUSE/REQUIRE, ACCOUNT LOCK 비밀번호 정책. `#MySQL #admin #user` `DBMS/MySQL/admin/`
- [[2026-06-13-48_(MySQL-권한-관리-체계)]] : 권한 범위(전역/DB/객체)·특성(Static/Dynamic)·역할(관리자/백업/서비스). GRANT 예시, 도구별 백업계정 최소권한. `#MySQL #admin #권한` `DBMS/MySQL/admin/`
- [[2026-06-13-49_(MySQL-sys-schema-접근-권한-부여)]] : 일반 유저 sys schema 사용 위한 SELECT *.* + sys.* EXECUTE + sys_config INSERT/UPDATE 권한. `#MySQL #admin #sys_schema` `DBMS/MySQL/admin/`
- [[2026-06-13-50_(MySQL-특정-유저-접속-차단)]] : 계정 삭제 없이 차단 5가지 — mysql_no_login, Host변경, 권한회수, AFTER CONNECT 트리거, my.cnf. `#MySQL #admin #접속차단` `DBMS/MySQL/admin/`
- [[2026-06-13-51_(MySQL-기-실행된-쿼리-내역-확인)]] : events_statements_history_long 컨슈머 활성화 후 processlist_id별 과거/현재 SQL_TEXT 추적. `#MySQL #admin #performance_schema` `DBMS/MySQL/admin/`
- [[2026-06-13-52_(MySQL-오래-수행중인-TX-찾기)]] : History List Length(undo) 개념·문제, 장시간/미커밋 트랜잭션·대량변경 쿼리 조회(sys.session, INNODB_TRX). `#MySQL #admin #트랜잭션` `DBMS/MySQL/admin/`
- [[2026-06-13-53_(MySQL-에러-유발-클라이언트-프로세스-찾기)]] : events_errors_summary→socket_instances→netstat로 에러 유발 세션의 IP·port·OS 프로세스 역추적. `#MySQL #admin #문제해결` `DBMS/MySQL/admin/`
- [[2026-06-13-54_(MySQL-쿼리-로깅-general-slow-log)]] : general_log/slow_query_log 켜기·log_output(FILE/TABLE), long_query_time, Percona 확장(rate_limit·global_control). `#MySQL #admin #로깅` `DBMS/MySQL/admin/`
- [[2026-06-13-55_(MySQL-블로킹-세션-조회-누가-누구를-막는지)]] : data_lock_waits 조인·sys.schema_table_lock_waits로 blocking↔waiting 세션 조회, sql_kill_blocking_connection, KILL 조치. `#MySQL #lock #blocking` `DBMS/MySQL/lock/`
- [[2026-06-13-56_(MySQL-Update-지연-사례-디스크-커밋-지연)]] : 단순 PK Update 지연 장애 — data_locks 전부 GRANTED, waiting for handler commit, iostat %util 포화로 락 아닌 디스크 커밋 지연. `#MySQL #lock #장애분석` `DBMS/MySQL/lock/`
- [[2026-06-13-57_(MySQL-Update-지연-사례-레코드락-commit-지연)]] : Update 지연2 — Lock_time≈20s, X,REC_NOT_GAP 레코드락, 다중 서버 중복 실행+앱 webhook commit 지연. `#MySQL #lock #장애분석` `DBMS/MySQL/lock/`
- [[2026-06-13-58_(MySQL-멀티-소스-복제-구성)]] : 다중 소스→1 replica, 채널별 CHANGE MASTER FOR CHANNEL, GTID 합산 gtid_purged, semi-sync 불가(Bug#102994). `#MySQL #replication #multi_source` `DBMS/MySQL/replication/`
- [[2026-06-13-59_(MySQL-Slave-gtid_executed-Master와-일치시키기)]] : RESET MASTER→gtid_purged 수동설정→AUTO_POSITION 재연결로 Slave GTID 동기화. `#MySQL #replication #GTID` `DBMS/MySQL/replication/`
- [[2026-06-13-60_(MySQL-파티션-관리)]] : RANGE 파티션 생성·조회, MAXVALUE REORGANIZE로 구간 추가, DROP/REMOVE PARTITIONING, EXCHANGE PARTITION 테이블 교체. `#MySQL #partition` `DBMS/MySQL/`
- [[2026-06-13-61_(MySQL-Timezone-설정)]] : CONVERT_TZ NULL·ERROR 1298 원인은 mysql.time_zone 테이블 비어있음. mysql_tzinfo_to_sql로 zoneinfo 적재. `#MySQL #timezone` `DBMS/MySQL/`
- [[2026-06-13-62_(MySQL-데이터-Load-Bulk-Insert-성능-개선)]] : 대량 적재 8기법 — binlog/autocommit/제약 OFF, multi-value INSERT, PK순서, LOAD DATA(20배), flush 설정. `#MySQL #bulk_insert #성능` `DBMS/쿼리튜닝/`
- [[2026-06-13-63_(MySQL-성능-최적화-커널-파라미터)]] : Linux sysctl(file-max·swappiness·dirty·net backlog·aio) + limits.conf nofile MySQL 서버 튜닝. `#MySQL #install #kernel` `DBMS/MySQL/installation/`
- [[2026-06-13-64_(MySQL-증분-백업-자동화-스크립트-LSN기반)]] : lastbackupinfo로 last_lsn 전달하는 증분백업 스크립트, 복구 순서(마지막만 롤백), PXB-2406 버그(8.0.23-16+). `#MySQL #backup #script` `DBMS/MySQL/backup/`
- [[2026-06-02_(Linux-고정-IP-설정)]] : CentOS/RHEL ifcfg 파일로 고정 IP 설정, nmcli 강제 적용, DHCP 덮어쓰기 방지 `#Linux #네트워크 #IP고정` `시스템/`

---

## 📥 원본 처리 현황

> raw/ 원본 → wiki 문서 처리 추적. **여기 등록된 원본 = 처리 완료.**
> `/ingest` 시 raw/ 스캔 결과(attachments·pdf2md 제외)에서 이 목록에 **없는 파일 = 미처리**.
> ✅ 처리 완료 파일: `raw/archived/{YYYY}/{카테고리}/` 로 자동 이동 (`.claudeignore` 지정).

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

### raw/clippings/mysql/ (MySQL, 2026-06-12 · 71건) ✅ 처리 완료 (2026-06-13)

**→ [[2026-06-13_(MySQL-Lock-Deadlock-모니터링)]]**
- `2026-06-12-mysql lock - BigDataTeam.md`, `2026-06-12-mysql deadlock - BigDataTeam.md`, `2026-06-12-Lock 개념 - BigDataTeam.md`, `2026-06-12-누가누구를 막고 있는지 보기 - BigDataTeam.md`, `2026-06-12-meta lock 을 잡고 있는 세션찾기 - BigDataTeam.md`, `2026-06-12-단순 Update문 지연사례 분석 - BigDataTeam.md`, `2026-06-12-Update 문 지연2 - BigDataTeam.md`, `2026-06-12-mysql commit 안하고 있는 세션찾기 - BigDataTeam.md`

**→ [[2026-06-13_(MySQL-Replication-가이드)]]**
- `2026-06-12-01. binary log file position based replication - BigDataTeam.md`, `2026-06-12-02. replication with global transaction identifiers - BigDataTeam.md`, `2026-06-12-03. semi sync replication - BigDataTeam.md`, `2026-06-12-05. replcation command 정리 - BigDataTeam.md`, `2026-06-12-07. replication 상황별 조치 - BigDataTeam.md`, `2026-06-12-semi replication - BigDataTeam.md`, `2026-06-12-replication 재설정 방법 - BigDataTeam.md`, `2026-06-12-slave 추가방법(cone statement) - BigDataTeam.md`, `2026-06-12-slave 추가방법(gtid based) - BigDataTeam.md`, `2026-06-12-slave 추가방법(mysqldump) - BigDataTeam.md`, `2026-06-12-slave 의 gtid_executed 를 master와 일치시키기 - BigDataTeam.md`, `2026-06-12-멀티 소스 복제 구성 - BigDataTeam.md`, `2026-06-12-BOS DB HA(failover) 테스트 - BigDataTeam.md`

**→ [[2026-06-13_(MySQL-XtraBackup-백업복구-가이드)]]**
- `2026-06-12-Xtrabackup Install setting - BigDataTeam.md`, `2026-06-12-XtraBackup의 동작원리 - BigDataTeam.md`, `2026-06-12-Xtrabackup 백업 요소 - BigDataTeam.md`, `2026-06-12-Full 백업 및 복구 - BigDataTeam.md`, `2026-06-12-증분 백업 - BigDataTeam.md`, `2026-06-12-증분 백업 복구 테스트 - BigDataTeam.md`, `2026-06-12-증분백업script(V8) - BigDataTeam.md`, `2026-06-12-full백업script ( V8 ) - BigDataTeam.md`, `2026-06-12-full백업script( v2.4 구버전용) - BigDataTeam.md`, `2026-06-12-백업 복구 시나리오 - BigDataTeam.md`, `2026-06-12-백업 복구 테스트 - BigDataTeam.md`, `2026-06-12-시점 복구 - BigDataTeam.md`, `2026-06-12-시점 복구 테스트 - BigDataTeam.md`, `2026-06-12-긴급장애 복구 - BigDataTeam.md`

**→ [[2026-06-13_(MySQL-Percona-설치-가이드)]]**
- `2026-06-12-install percona by rpm on redhat - BigDataTeam.md`, `2026-06-12-install percona by rpm on redhat(설치요약본) - BigDataTeam.md`, `2026-06-12-percona mysql devel 환경만들기 - BigDataTeam.md`, `2026-06-12-Percona MySQL 경로 수정 - BigDataTeam.md`, `2026-06-12-percona server 8.0.40 패치과정 - BigDataTeam.md`, `2026-06-12-percona mysql 8.4  vs percona mysql 8.0 - BigDataTeam.md`, `2026-06-12-prd db용 my.cnf - BigDataTeam.md`, `2026-06-12-성능 최적화를 위한 주요 커널파라미터 - BigDataTeam.md`

**→ [[2026-06-13_(MySQL-Performance-Schema-활용)]]**
- `2026-06-12-MySQL Performance_schema 활용 - BigDataTeam.md`, `2026-06-12-Performance 스키마를 이용한 프로파일링 - BigDataTeam.md`, `2026-06-12-events_statements_summary_by_digest를 이용하여 SQL성능 분석 - BigDataTeam.md`, `2026-06-12-MySQL 성능 튜닝 - BigDataTeam.md`, `2026-06-12-mysql optimization Index - BigDataTeam.md`, `2026-06-12-fast index creation - BigDataTeam.md`, `2026-06-12-데이터 Load 성능 올리기 - BigDataTeam.md`, `2026-06-12-mymon shell script for monitoring MySQL.md`

**→ [[2026-06-13_(MySQL-관리자-쿼리-모음)]]**
- `2026-06-12-mysql user 생성 - BigDataTeam.md`, `2026-06-12-mysql 권한 관리 - BigDataTeam.md`, `2026-06-12-sys schema 접근 권한 부여 - BigDataTeam.md`, `2026-06-12-MySQL Audit - BigDataTeam.md`, `2026-06-12-특정유저의 접속차단 - BigDataTeam.md`, `2026-06-12-03. MySQL admin 쿼리 - BigDataTeam.md`, `2026-06-12-기 실행된 쿼리내역 확인 - BigDataTeam.md`, `2026-06-12-오래 수행중인 TX 찾기 - BigDataTeam.md`, `2026-06-12-에러를 일으키는 client 프로세스 찾기 - BigDataTeam.md`, `2026-06-12-쿼리 로깅하기 - BigDataTeam.md`

### raw/archived/2026/MySQL/clippings/ (1:1 개별 재처리 — 배치 6~)

**배치 6 (InnoDB 구조, 2026-06-13)** — 5개
- `2026-06-12-리두로그 및 로그버퍼 - BigDataTeam.md` → [[2026-06-13-26_(MySQL-Redo-Log-및-로그버퍼-설정)]]
- `2026-06-12-리두로그 아카이빙 - BigDataTeam.md` → [[2026-06-13-27_(MySQL-Redo-Log-아카이빙)]]
- `2026-06-12-리두로그 활성화 및 비활성화 - BigDataTeam.md` → [[2026-06-13-28_(MySQL-Redo-Log-활성화-비활성화)]]
- `2026-06-12-mysql undo 관리 - BigDataTeam.md` → [[2026-06-13-29_(MySQL-Undo-로그-관리)]]
- `2026-06-12-체인지버퍼 - BigDataTeam.md` → [[2026-06-13-30_(MySQL-Change-Buffer)]]

**배치 7 (Replication 나머지, 2026-06-13)** — 5개 (4 신규 + 1 병합)
- `2026-06-12-slave 추가방법(cone statement) - BigDataTeam.md` → [[2026-06-13-31_(MySQL-Slave-추가-CLONE-플러그인)]]
- `2026-06-12-slave 추가방법(gtid based) - BigDataTeam.md` → [[2026-06-13-32_(MySQL-Slave-추가-XtraBackup-GTID)]]
- `2026-06-12-slave 추가방법(mysqldump) - BigDataTeam.md` → [[2026-06-13-33_(MySQL-Slave-추가-mysqldump)]]
- `2026-06-12-replication 재설정 방법 - BigDataTeam.md` → [[2026-06-13-34_(MySQL-Replication-재설정)]]
- `2026-06-12-semi replication - BigDataTeam.md` → [[2026-06-13-03_(MySQL-Semi-Sync-복제)]] (기존 문서에 병합)

**배치 8 (Backup/XtraBackup 나머지, 2026-06-13)** — 5개 (4 신규, 증분 2건 병합)
- `2026-06-12-XtraBackup의 동작원리 - BigDataTeam.md` → [[2026-06-13-35_(MySQL-XtraBackup-동작원리)]]
- `2026-06-12-Xtrabackup 백업 요소 - BigDataTeam.md` → [[2026-06-13-36_(MySQL-XtraBackup-백업-요소-및-옵션)]]
- `2026-06-12-Xtrabackup Install setting - BigDataTeam.md` → [[2026-06-13-37_(MySQL-XtraBackup-설치-및-백업-계정-권한)]]
- `2026-06-12-증분 백업 - BigDataTeam.md` → [[2026-06-13-38_(MySQL-XtraBackup-증분-백업-및-복구)]]
- `2026-06-12-증분 백업 복구 테스트 - BigDataTeam.md` → [[2026-06-13-38_(MySQL-XtraBackup-증분-백업-및-복구)]] (자동화 섹션 병합)

**배치 9 (Percona/설치 나머지, 2026-06-13)** — 5개
- `2026-06-12-percona mysql 8.4  vs percona mysql 8.0 - BigDataTeam.md` → [[2026-06-13-39_(Percona-MySQL-8.4-vs-8.0-비교)]]
- `2026-06-12-percona mysql devel 환경만들기 - BigDataTeam.md` → [[2026-06-13-40_(Percona-MySQL-devel-환경-구성)]]
- `2026-06-12-Percona MySQL 경로 수정 - BigDataTeam.md` → [[2026-06-13-41_(Percona-MySQL-datadir-basedir-경로-변경)]]
- `2026-06-12-percona server 8.0.40 패치과정 - BigDataTeam.md` → [[2026-06-13-42_(Percona-Server-8.0.40-패치과정)]]
- `2026-06-12-prd db용 my.cnf - BigDataTeam.md` → [[2026-06-13-43_(Percona-MySQL-운영-표준-my.cnf)]]

**배치 10 (백업/복구 시나리오, 2026-06-13)** — 5개 (3 신규, 테스트 2건 병합)
- `2026-06-12-시점 복구 - BigDataTeam.md` → [[2026-06-13-44_(MySQL-시점-복구-PITR)]]
- `2026-06-12-시점 복구 테스트 - BigDataTeam.md` → [[2026-06-13-44_(MySQL-시점-복구-PITR)]] (테스트 검증 섹션 병합)
- `2026-06-12-백업 복구 시나리오 - BigDataTeam.md` → [[2026-06-13-45_(MySQL-백업-복구-시나리오)]]
- `2026-06-12-백업 복구 테스트 - BigDataTeam.md` → [[2026-06-13-45_(MySQL-백업-복구-시나리오)]] (qpress 압축복구 검증 섹션 병합)
- `2026-06-12-긴급장애 복구 - BigDataTeam.md` → [[2026-06-13-46_(MySQL-긴급장애-복구-innodb_force_recovery)]]

**배치 11 (권한/계정 관리, 2026-06-13)** — 4개
- `2026-06-12-mysql user 생성 - BigDataTeam.md` → [[2026-06-13-47_(MySQL-User-계정-생성-옵션)]]
- `2026-06-12-mysql 권한 관리 - BigDataTeam.md` → [[2026-06-13-48_(MySQL-권한-관리-체계)]]
- `2026-06-12-sys schema 접근 권한 부여 - BigDataTeam.md` → [[2026-06-13-49_(MySQL-sys-schema-접근-권한-부여)]]
- `2026-06-12-특정유저의 접속차단 - BigDataTeam.md` → [[2026-06-13-50_(MySQL-특정-유저-접속-차단)]]

**배치 12 (admin 조회/진단, 2026-06-13)** — 4개
- `2026-06-12-기 실행된 쿼리내역 확인 - BigDataTeam.md` → [[2026-06-13-51_(MySQL-기-실행된-쿼리-내역-확인)]]
- `2026-06-12-오래 수행중인 TX 찾기 - BigDataTeam.md` → [[2026-06-13-52_(MySQL-오래-수행중인-TX-찾기)]]
- `2026-06-12-에러를 일으키는 client 프로세스 찾기 - BigDataTeam.md` → [[2026-06-13-53_(MySQL-에러-유발-클라이언트-프로세스-찾기)]]
- `2026-06-12-쿼리 로깅하기 - BigDataTeam.md` → [[2026-06-13-54_(MySQL-쿼리-로깅-general-slow-log)]]

**배치 13 (Lock 지연사례 + Replication, 2026-06-13)** — 5개
- `2026-06-12-누가누구를 막고 있는지 보기 - BigDataTeam.md` → [[2026-06-13-55_(MySQL-블로킹-세션-조회-누가-누구를-막는지)]]
- `2026-06-12-단순 Update문 지연사례 분석 - BigDataTeam.md` → [[2026-06-13-56_(MySQL-Update-지연-사례-디스크-커밋-지연)]]
- `2026-06-12-Update 문 지연2 - BigDataTeam.md` → [[2026-06-13-57_(MySQL-Update-지연-사례-레코드락-commit-지연)]]
- `2026-06-12-멀티 소스 복제 구성 - BigDataTeam.md` → [[2026-06-13-58_(MySQL-멀티-소스-복제-구성)]]
- `2026-06-12-slave 의 gtid_executed 를 master와 일치시키기 - BigDataTeam.md` → [[2026-06-13-59_(MySQL-Slave-gtid_executed-Master와-일치시키기)]]

**배치 14 (Schema·성능·커널·백업스크립트 + Performance 중복정리, 2026-06-13)** — 8개 (5 신규, 3 기존 Performance 문서 커버)
- `2026-06-12-mysql partition 관리 - BigDataTeam.md` → [[2026-06-13-60_(MySQL-파티션-관리)]]
- `2026-06-12-MySQL timezone 설정 - BigDataTeam.md` → [[2026-06-13-61_(MySQL-Timezone-설정)]]
- `2026-06-12-데이터 Load 성능 올리기 - BigDataTeam.md` → [[2026-06-13-62_(MySQL-데이터-Load-Bulk-Insert-성능-개선)]]
- `2026-06-12-성능 최적화를 위한 주요 커널파라미터 - BigDataTeam.md` → [[2026-06-13-63_(MySQL-성능-최적화-커널-파라미터)]]
- `2026-06-12-증분백업script(V8) - BigDataTeam.md` → [[2026-06-13-64_(MySQL-증분-백업-자동화-스크립트-LSN기반)]]
- `2026-06-12-MySQL Performance_schema 활용 - BigDataTeam.md` → [[2026-06-13_(MySQL-Performance-Schema-활용)]] (2.5단계: 제목 90%+ 일치, 기존 문서가 이미 커버 — 중복 생성 금지)
- `2026-06-12-MySQL 성능 튜닝 - BigDataTeam.md` → [[2026-06-13_(MySQL-Performance-Schema-활용)]] (기존 문서 참조 원본, 커버됨)
- `2026-06-12-Performance 스키마를 이용한 프로파일링 - BigDataTeam.md` → [[2026-06-13_(MySQL-Performance-Schema-활용)]] (기존 문서 참조 원본, 커버됨)
