# MySQL MY-013360 — mysql_native_password Deprecated 경고

- **카테고리**: #DBMS #MySQL #troubleshooting
- **태그**: #MySQL #troubleshooting #error #authentication #caching_sha2_password #proxysql
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-MY-013360 mysql_native_password' is deprecated 오류 - BigDataTeam]]

## 1. 핵심 요약

MySQL 8.0.34+에서 `mysql_native_password`가 deprecated되어 에러 로그에 `[Warning] [MY-013360]`이 반복 출력된다. 근본 해결은 **유저 인증 방식을 `caching_sha2_password`로 전환**하는 것이지만, ProxySQL 2.6.0 미만은 sha2를 미지원하므로 버전을 함께 확인해야 한다.

---

## 2. 해결 방법 (3가지)

### 방법 1 — 로그 레벨 낮추기 (임시)
```sql
SET GLOBAL log_error_verbosity = 1;
FLUSH ERROR LOGS;
```
> WARNING이 전부 숨겨지는 부작용. 임시 방편.

### 방법 2 — 특정 에러코드만 suppress (권장 임시)
```ini
[mysqld]
log_error_suppression_list='MY-013360'
```

### 방법 3 — 인증 방식 변경 (근본 해결)
```ini
# my.cnf에서 제거
# default_authentication_plugin=mysql_native_password
```
```sql
ALTER USER 'the_user'@'the_host' IDENTIFIED WITH caching_sha2_password BY 'new_password';
```

## 3. ProxySQL 연동 시 주의

- ProxySQL **2.6.0+** 부터 `caching_sha2_password` 지원
- 하위 버전에서는 ProxySQL 경유 유저를 `mysql_native_password`로 유지해야 함
- ProxySQL에서 sha2 설정:
  ```sql
  SET mysql-default_authentication_plugin = 'caching_sha2_password';
  LOAD MYSQL VARIABLES TO RUNTIME; SAVE MYSQL VARIABLES TO DISK;
  ```

## 4. 연관 개념

- [[2026-06-14-MS36_(MySQL-MY-013712-keyring_component_metadata_query-Warning)]]
- [[2026-06-13-47_(MySQL-User-계정-생성-옵션)]]
