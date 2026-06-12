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
