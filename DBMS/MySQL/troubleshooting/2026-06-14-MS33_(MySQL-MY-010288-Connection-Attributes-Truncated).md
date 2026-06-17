# MySQL MY-010288 — Connection Attributes Truncated

- **카테고리**: #DBMS #MySQL #troubleshooting
- **태그**: #MySQL #troubleshooting #error #proxysql #performance_schema #connection-attrs
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-MY-010288 - BigDataTeam]]

## 1. 핵심 요약

`[Warning] [MY-010288] Connection attributes of length 566 were truncated (72 bytes lost)`는 클라이언트(ProxySQL 등)가 `performance_schema_session_connect_attrs_size` 한도보다 긴 연결 속성을 보낼 때 발생하는 경고. `performance_schema_session_connect_attrs_size`를 충분히 늘리면 해소된다.

---

## 2. 원인

- 클라이언트는 접속 시 `CLIENT_CONNECT_ATTRS`로 key-value 세션 정보를 전달
- ProxySQL이 MySQL 버그 workaround 시도 중 속성 크기가 `performance_schema_session_connect_attrs_size`(기본 512) 초과 → truncated + Warning
- 초과분은 손실되며 Warning만 출력됨(서비스에는 영향 없음)

## 3. 해결

```sql
-- 현재 값 확인
SHOW VARIABLES LIKE 'performance_schema_session_connect_attrs_size';
-- 기본 512, 커넥션 속성 길이(예시: 566)보다 크게 설정
SET GLOBAL performance_schema_session_connect_attrs_size = 1024;

-- 영구 적용 (my.cnf)
[mysqld]
performance_schema_session_connect_attrs_size = 1024
```

## 4. 연관 개념

- [[2026-06-14-MS34_(MySQL-MY-011370-011355-010202-keyring_file-오류)]]
