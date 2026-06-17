# MySQL MY-013712 — keyring_component_metadata_query Warning

- **카테고리**: #DBMS #MySQL #troubleshooting
- **태그**: #MySQL #troubleshooting #error #keyring #warning #log_suppression
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-MY-013712 No suitable 'keyring_component_metadata_query' .. - BigDataTeam]]

## 1. 핵심 요약

`[Warning] [MY-013712] No suitable 'keyring_component_metadata_query' service implementation found`는 **MySQL 버그(#103684)** 로, 근본 해결은 버그 픽스를 기다리는 것 외에 방법이 없다. 그 전까지는 `log_error_suppression_list`로 해당 경고를 억제한다.

---

## 2. 에러

```
2024-06-28T11:50:02 14278 [Warning] [MY-013712] [Server]
No suitable 'keyring_component_metadata_query' service implementation found to fulfill the request.
```

- MySQL 버그 리포트: https://bugs.mysql.com/bug.php?id=103684

## 3. 조치 — 경고 억제 (버그 픽스 전 임시)

```sql
-- 세션
SET GLOBAL log_error_suppression_list = 'MY-013360,MY-013712';
```
```ini
-- my.cnf (영구)
[mysqld]
log_error_suppression_list='MY-013360,MY-013712'
```
> MY-013360(mysql_native_password deprecated)과 함께 억제하는 경우가 많다.

## 4. 연관 개념

- [[2026-06-14-MS35_(MySQL-MY-013360-mysql_native_password-deprecated)]] — 함께 suppress
- [[2026-06-14-MS34_(MySQL-MY-011370-011355-010202-keyring_file-오류)]] — keyring 관련 에러
