# PostgreSQL psql 메타명령어 레퍼런스

- **카테고리**: #DBMS #PostgreSQL #admin
- **태그**: #PostgreSQL #psql #메타명령어 #backslash
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-psql 명령어 - BigDataTeam]], [[2026-06-14-postgresql 참고자료 - BigDataTeam]]

## 1. 핵심 요약

psql의 백슬래시(`\`) 메타명령어로 객체 조회(`\d`·`\dt`·`\du`·`\l`), DB 접속(`\c`), 편집(`\e`), 변수(`\set`)를 수행합니다.
`\?`(메타명령 도움말), `\h`(SQL 문법 도움말)가 출발점입니다.

---

## 2. 도움말

```
\?            -- 백슬래시 명령 도움말
\h [NAME]     -- SQL 문법 도움말
```

## 3. 객체 조회

| 명령 | 내용 |
|------|------|
| `\du` | role(사용자) 목록·속성 (= `SELECT * FROM pg_shadow`) |
| `\l` | database 목록 (= `SELECT * FROM pg_database`) |
| `\d` | relation 목록 / `\d+` 상세(크기 포함) |
| `\dt` / `\dv` / `\dS` | 테이블 / 뷰 / 시스템 테이블 목록 |
| `\dp` | 객체 access privileges (→ [[2026-06-14-PG25_(PostgreSQL-Access-Privileges-표기-해석)]]) |

## 4. 접속 / 세션

```
\c bos                  -- database 전환
\c dvdrental postgres   -- DB+user 지정 접속
\q                      -- psql 종료
```

## 5. 편집 / 실행 / 변수

```
\e                      -- 임시 SQL을 vi로 편집 후 실행
\g                      -- 직전 SQL/명령 재실행 (\gx = expanded)
\watch [SEC]            -- SEC초마다 반복 실행
\s                      -- 명령 history 출력
\set city seoul         -- 변수 선언 → \echo :city
```
- Special 변수: `AUTOCOMMIT`, `ON_ERROR_STOP`, `ON_ERROR_ROLLBACK`, `ENCODING`, `VERBOSITY`, `PROMPT1` 등.
  ```
  \set AUTOCOMMIT off
  \set ON_ERROR_STOP on
  ```

## 6. 추가 학습 자료 (PDF)

> `2026-06-14-postgresql 참고자료`의 첨부 자료 — 단독 문서 가치가 낮아 본 문서에 통합.

- PostgreSQL & PPAS 실무, postgresql_튜닝교육자료, 실행통계 활용(pptx) 등 PDF/PPTX 학습 자료.

## 7. 연관 개념

- [[2026-06-02_(PostgreSQL-운영-유틸리티-모음)]]
- [[2026-06-14-PG25_(PostgreSQL-Access-Privileges-표기-해석)]]
