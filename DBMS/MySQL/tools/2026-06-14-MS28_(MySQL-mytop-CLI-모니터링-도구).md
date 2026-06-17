# MySQL mytop (CLI 실시간 모니터링 도구)

- **카테고리**: #DBMS #MySQL #tools
- **태그**: #MySQL #tools #mytop #monitoring #processlist #CLI
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-mytop - BigDataTeam]]

## 1. 핵심 요약

`mytop`은 MySQL 전용 `top` 유사 CLI 모니터링 도구로, 실시간 쿼리 목록·스레드 상태·InnoDB 상태·초당 쿼리 수를 터미널에서 확인한다. `yum install mytop`으로 설치하며, 인터랙티브 키로 스레드 kill·DB 필터·InnoDB 상태 전환이 가능하다.

---

## 2. 설치·실행

```bash
yum install mytop
mytop -uroot -p -P 3306 -d BOSRL
```

## 3. 주요 인터랙티브 키

| 키 | 기능 |
|----|------|
| `t` | 스레드 목록(기본) |
| `I` | InnoDB 상태 |
| `c` | 명령 요약(Com_* 카운터) |
| `m` | qps 스크롤 뷰 |
| `k` | 스레드 kill |
| `d` | 특정 DB만 표시 |
| `u` | 특정 user만 표시 |
| `h` | 특정 host만 표시 |
| `i` | idle(sleeping) 스레드 숨김 토글 |
| `e` | 해당 스레드 실행 쿼리 EXPLAIN |
| `f` | 해당 스레드 풀 쿼리 |
| `q` | 종료 |

## 4. 연관 개념

- [[2026-06-14-MS29_(MySQL-Innotop-InnoDB-CLI-모니터링-도구)]]
- [[2026-06-13-24_(MySQL-InnoDB-Lock-모니터링)]]
