# MySQL CLI prompt 커스텀

- **카테고리**: #DBMS #MySQL #tools
- **태그**: #MySQL #tools #mysql-cli #prompt #CLI설정
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-prompt - BigDataTeam]]

## 1. 핵심 요약

MySQL CLI 프롬프트를 `(\u@\h) [\d]>` 형식으로 커스텀하면 **현재 접속 계정·호스트·DB**를 한눈에 볼 수 있어 다중 서버 작업 실수를 방지한다. 환경변수·옵션·`.my.cnf`·세션 내 `prompt` 명령 4가지 방법으로 설정 가능.

---

## 2. 설정 방법

```bash
# 방법 1: 환경변수
export MYSQL_PS1="(\u@\h) [\d]> "

# 방법 2: 접속 옵션
mysql --prompt="(\u@\h) [\d]> "

# 방법 3: ~/.my.cnf 영구 설정
[mysql]
prompt=(\\u@\\h) [\\d]>\\_

# 방법 4: 세션 내 변경
mysql> prompt (\u@\h) [\d]>\_
mysql> prompt             ← 기본값 복원
```

## 3. 주요 옵션

| 변수 | 의미 | | 변수 | 의미 |
|------|------|-|------|------|
| `\u` | 사용자명 | | `\h` | 서버 호스트 |
| `\d` | 현재 DB | | `\v` | 서버 버전 |
| `\D` | 전체 날짜 | | `\p` | 포트/소켓 |
| `\R` | 24시간 시각 | | `\_` | 스페이스 |
| `\c` | 명령 카운터 | | `\n` | 줄바꿈 |

## 4. 연관 개념

- [[2026-06-14-MS25_(MySQL-CLI-auto-rehash-탭완성-프로시저-문제)]]
- [[2026-06-14-MS26_(MySQL-mysql_config_editor-login-path)]]
