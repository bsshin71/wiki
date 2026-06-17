# MySQL mysqlpump (병렬 덤프 도구)

- **카테고리**: #DBMS #MySQL #tools
- **태그**: #MySQL #tools #mysqlpump #mysqldump #backup #병렬처리
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-mysqlpump - BigDataTeam]]

## 1. 핵심 요약

`mysqlpump`는 `mysqldump`와 유사하지만 **병렬 처리**를 지원해 대용량 export/import가 더 빠르며, **DB include/exclude 선택**과 **mysql.user 테이블 백업**이 가능하다. 단, **MySQL 8.4.x에서 Deprecated** 예정이므로 장기 사용 시 주의.

---

## 2. 특징

| 항목 | mysqlpump | mysqldump |
|------|-----------|-----------|
| 병렬처리 | ✅ 가능 | ❌ 단일 스레드 |
| DB include/exclude | ✅ 지원 | 제한적 |
| mysql.user 백업 | ✅ 가능 | ❌ 미포함(기본) |
| 8.4+ 지원 | ⚠️ Deprecated 예정 | ✅ 계속 지원 |

## 3. 사용 예

```bash
# mysql.user 정보 export (유저+권한 함께 내보내기)
mysqlpump --user=root --password='' \
  --exclude-databases=% \
  --set-gtid-purged=OFF \
  --users > user.sql
```
- `--exclude-databases=%`: 모든 DB 제외(유저 정보만 export 시)
- `--users`: mysql.user 포함(CREATE USER + GRANT 함께 출력)

## 4. 연관 개념

- [[2026-06-14-MS28_(MySQL-mytop-CLI-모니터링-도구)]]
- [[2026-06-13-36_(MySQL-XtraBackup-백업-요소-및-옵션)]]
