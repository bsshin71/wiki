# PostgreSQL Access Privileges 표기 해석

- **카테고리**: #DBMS #PostgreSQL #admin
- **태그**: #PostgreSQL #권한 #access_privileges #aclitem
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-13-access privileges 해석 - BigDataTeam]]

## 1. 핵심 요약

`\l`·`\dp`의 **Access privileges** 컬럼은 **`<user>=<권한문자>/<부여자>`** 형식이며, user가 생략된 `=c`는 PUBLIC을 의미합니다.
권한 문자는 r(SELECT)·w(UPDATE)·a(INSERT)·d(DELETE)·C(CREATE)·c(CONNECT)·T(TEMPORARY) 등으로 구성됩니다.

---

## 2. 표기 형식

```
<user> = <permissions> / <granted by>
=c/postgres            → PUBLIC에게 CONNECT 부여(postgres가 부여)
postgres=CTc/postgres  → postgres에게 CREATE+TEMP+CONNECT
scott=arwdDxt/scott    → scott에게 테이블 ALL PRIVILEGES
```
> `user` 생략(`=`)은 **PUBLIC**, `*`는 grant option, `/yyyy`는 부여한 role.

## 3. 권한 문자

| 문자 | 권한 | 문자 | 권한 |
|------|------|------|------|
| r | SELECT (read) | C | CREATE |
| w | UPDATE (write) | c | CONNECT |
| a | INSERT (append) | T | TEMPORARY |
| d | DELETE | X | EXECUTE |
| D | TRUNCATE | U | USAGE |
| x | REFERENCES | t | TRIGGER |

- **테이블 ALL** = `arwdDxt`
- **DATABASE** 기본: `=c/postgres`(PUBLIC CONNECT) + `owner=CTc`

## 4. 예시 (\dp)

```
 Schema |  Name  | Type  |  Access privileges
--------+--------+-------+---------------------
 public | bonus  | table |                       ← 권한 표기 없음 = owner 기본권한만
 scott  | test   | table | scott=arwdDxt/scott   ← scott에게 ALL
```
> 빈 칸은 명시적 GRANT가 없음(소유자 기본 권한만 존재)을 의미.

## 5. 연관 개념

- [[2026-06-14-PG07_(PostgreSQL-권한-관리-GRANT)]]
- [[2026-06-14-PG24_(PostgreSQL-psql-메타명령어-레퍼런스)]]
