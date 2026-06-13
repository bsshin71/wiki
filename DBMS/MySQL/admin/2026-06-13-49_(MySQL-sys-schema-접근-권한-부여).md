# MySQL sys schema 접근 권한 부여

- **카테고리**: #DBMS #MySQL #admin
- **태그**: #admin #권한 #sys_schema #모니터링
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-sys schema 접근 권한 부여 - BigDataTeam]]

## 1. 핵심 요약

root가 아닌 일반(모니터링) 유저가 sys schema를 사용하려면 **전역 SELECT + sys 스키마 EXECUTE + sys_config 테이블 INSERT/UPDATE** 권한이 필요합니다.

---

## 2. 부여할 권한

```sql
grant select on *.* to `mon`@`192.168.%`;
grant execute on sys.* to `mon`@`192.168.%`;
grant insert, update on sys.sys_config to `mon`@`192.168.%`;
```

| 권한 | 용도 |
|------|------|
| `SELECT ON *.*` | sys 뷰가 참조하는 performance_schema·information_schema 조회 |
| `EXECUTE ON sys.*` | sys 스키마의 저장 프로시저/함수 실행 |
| `INSERT, UPDATE ON sys.sys_config` | sys 동작 설정값(`sys_config`) 변경 |

> 참고: https://dev.mysql.com/doc/refman/8.4/en/sys-schema-prerequisites.html

---

## 3. 연관 개념

- [[2026-06-13-48_(MySQL-권한-관리-체계)]]
- [[2026-06-13_(MySQL-Performance-Schema-활용)]]
