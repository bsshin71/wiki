# Percona MySQL devel 환경 구성

- **카테고리**: #DBMS #MySQL #Installation
- **태그**: #install #devel #client환경 #libmysqlclient
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-percona mysql devel 환경만들기 - BigDataTeam]]

## 1. 핵심 요약

C/C++ 프로그램에서 MySQL에 접속하려면 헤더파일·클라이언트 라이브러리(devel·shared·shared-compat·client)를 설치해야 합니다.
**헤더파일 경로 참조 버그(Bug #92870)가 수정된 8.0.21 이상**을 설치해야 하며, 기존 `mariadb` 라이브러리가 있으면 충돌하므로 먼저 제거합니다.

---

## 2. 필요 패키지

| package | 내용 |
|---------|------|
| percona-server-devel | 헤더 파일 (`mysql.h` 등) |
| percona-server-shared | 클라이언트 공유 라이브러리 |
| percona-server-shared-compat | 구버전 호환 라이브러리 (`libmysqlclient.so.18` 등) |
| percona-server-client | 커맨드라인 클라이언트(`mysql` tool) |

다운로드: https://www.percona.com/downloads/Percona-Server-LATEST/

---

## 3. 주의점 — 8.0.21 이상 설치

- 헤더 경로 참조 버그(`mysql/udf_registration_types.h: No such file`)가 8.0.21에서 수정됨.
- BUG: https://bugs.mysql.com/bug.php?id=92870

```cpp
// 8.0.21 미만에서 발생하는 컴파일 오류
/usr/include/mysql/mysql_com.h:1046:42: fatal error:
  mysql/udf_registration_types.h: No such file or directory
```

---

## 4. mariadb 라이브러리 제거 (선행 필수)

```bash
yum list installed | grep maria
yum remove mariadb.x86_64 mariadb-devel.x86_64 mariadb-libs.x86_64
```

---

## 5. 설치 확인

```bash
locate -b libmysqlclient*21*
# /usr/lib64/mysql/libmysqlclient.so.21   ← 라이브러리 설치 확인
/usr/bin/mysql --help | head
# Ver 8.0.22-13 ... Percona Server (GPL)   ← client tool 확인
```

---

## 6. 컴파일 테스트

```makefile
# Makefile
CC=gcc
CFLAGS=-g
LIBS=-L/usr/lib64/mysql -L/usr/lib64 -lmysqlclient -lpthread -lz -lm -lrt -lssl -lcrypto -ldl
OBJS=connect.o
TARGET=pconnect
all: $(TARGET)
$(TARGET): $(OBJS)
	$(CC) -o $@ $(OBJS) $(CFLAGS) $(INC) $(LIBS)
```

```c
// connect.c 핵심
#include <mysql/mysql.h>
conn = mysql_init(NULL);
mysql_real_connect(conn, server, user, password, database, 3306, NULL, 0);
mysql_query(conn, "show tables");
```

```bash
# 링크 확인
ldd ./pconnect
#  libmysqlclient.so.21 => /usr/lib64/mysql/libmysqlclient.so.21   ← 경로·버전 확인
```

---

## 7. 참고 — libmysqlclient vs libperconaserverclient

- `libmysqlclient.so`는 `libperconaserverclient.so`의 이름만 바뀐 것(Percona 5.6 기준).
- 단, Percona 8.0대에서는 두 라이브러리 크기 차이가 큼 → `libperconaserverclient.so`는 여러 라이브러리 기능을 하나로 통합한 것으로 보임.

---

## 8. 연관 개념

- [[2026-06-13-14_(MySQL-Percona-설치-상세-가이드)]]
