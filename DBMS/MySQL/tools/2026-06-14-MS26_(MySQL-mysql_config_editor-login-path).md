# MySQL mysql_config_editor (login-path 관리)

- **카테고리**: #DBMS #MySQL #tools
- **태그**: #MySQL #tools #mysql_config_editor #login-path #보안 #credentials
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-mysql_config_editor - BigDataTeam]]

## 1. 핵심 요약

`mysql_config_editor`로 DB 접속 정보(host·user·password·port)를 **암호화된 `~/.mylogin.cnf`** 에 저장하면, 커맨드라인에서 비밀번호를 노출 없이 접속할 수 있다(ssh config와 유사). `--login-path` 이름으로 여러 DB 연결 정보를 구분 관리한다.

---

## 2. 설정 (login-path 등록)

```bash
mysql_config_editor set -G client -h 호스트명 -u 계정명 -p
# -G: login-path 이름 (기본 'client')
# -p: 대화형으로 비밀번호 입력 (커맨드에 노출 안됨)
# 저장 위치: ~/.mylogin.cnf (암호화)
```

## 3. 접속

```bash
mysql --login-path=client           # 기본 'client' 프로파일 사용
mysql --login-path=shellclient      # 특정 프로파일 사용
```

## 4. 조회·삭제

```bash
# 조회
mysql_config_editor print -G shellclient    # 특정 login-path
mysql_config_editor print --all             # 전체

# 삭제
mysql_config_editor remove -G test -hPu    # 특정 옵션(h:host, P:port, u:user)
mysql_config_editor remove -G test         # login-path 전체
mysql_config_editor reset                   # 전체 초기화
```

## 5. 연관 개념

- [[2026-06-14-MS25_(MySQL-CLI-auto-rehash-탭완성-프로시저-문제)]]
- [[2026-06-14-MS31_(MySQL-CLI-prompt-커스텀)]]
