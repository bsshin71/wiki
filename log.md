# 📝 위키 업데이트 로그

- **2026-06-05**: 위키 시스템 구축 완료 — CLAUDE.md, wiki/index.md, wiki/log.md, .schema/ 파일 및 전체 폴더 구조 초기화.
- **2026-06-05**: `2026-06-05_(Oracle-유저-생성하기).md` 추가 — Oracle 19c 사용자 계정 생성 절차 (sysdba 접속, _ORACLE_SCRIPT 설정, CREATE USER, GRANT), ORA-65096 에러 대처법.
- **2026-06-05**: `2026-06-05_(Oracle-19c-Rocky-Linux-설치-가이드).md` 추가 — Rocky Linux 9.7 환경에서 Oracle 19.3.0.0 설치 시 GCC 11+ / libpthread 충돌 해결 전 과정 (심볼릭 링크 패치, ins_rdbms.mk 수정, 수동 relink).
- **2026-06-05**: `2026-06-05_(Oracle-리스너-등록).md` 추가 — Oracle 리스너 동적 등록(LREG 자동)과 정적 등록(listener.ora 수동) 차이 및 사용 시나리오 정리.
- **2026-06-05**: `wiki/DBMS/Oracle/install/` 서브폴더 생성 — #Oracle #install 문서 3개→4개 도달로 트리거. 기존 문서 3개 이동 및 `2026-06-05_(Oracle-DBCA-NETCA-가이드).md` 신규 추가.
- **2026-06-08**: `2026-06-08_(Advanced-SQL-PostgreSQL).md` 추가 — Oracle Advanced SQL 교육 과정 PostgreSQL 이식판. INDEX/IS NULL 차이, TOP-n, 조인·서브쿼리, ROLLUP·CUBE, 분석함수, 계층질의(WITH RECURSIVE), 정규식, Oracle→PG 이행 가이드 포함.
- **2026-06-08**: `2026-06-08_(Advanced-SQL-PostgreSQL).md` 재처리 — `--confirm` 강제 재처리 테스트. PDF 재변환(421,419 chars) 및 wiki 문서 덮어쓰기 완료.
- **2026-06-12**: `2026-06-05_(Claudian-Smart-Connections-MCP-설정가이드).md` 추가 — Obsidian+Claudian 환경 Smart Connections MCP 설정. nvm 심볼릭 링크 함정·Node v20 요구사항·mcp.json 직접 편집 방법 정리.
- **2026-06-12**: `2026-06-05_(Claudian-GitHub-MCP-설정가이드).md` 추가 — Obsidian+Claudian 환경 GitHub MCP 설정. PAT 발급·npx-cli.js 절대경로·보안 주의사항(mcp.json git 제외) 정리.
- **2026-06-12**: `2026-06-12_(LLMWiki-Graphify-통합-설정-로그).md` 추가 — graphify 도입 배경·설치·출력경로 문제 해결(robocopy 왕복)·wrapper BAT 3종·CLAUDE.md 변경사항·첫 /lint 결과 기록.
- **2026-06-12**: `2026-06-12_(Obsidian-Git-플러그인-연동가이드).md` 추가 — wiki/ 폴더만 GitHub Private 레포에 선택 동기화. Custom base path 설정·PAT 발급·첫 푸시 절차·자동 백업 설정.
