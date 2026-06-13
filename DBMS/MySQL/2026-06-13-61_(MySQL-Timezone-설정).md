# MySQL Timezone 설정 (time_zone 테이블 적재)

- **카테고리**: #DBMS #MySQL #Schema
- **태그**: #timezone #CONVERT_TZ #time_zone_name #장애조치
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-MySQL timezone 설정 - BigDataTeam]]

## 1. 핵심 요약

`CONVERT_TZ`에 'Asia/Seoul' 같은 **타임존 이름**을 쓰면 `NULL`이 반환되거나 `SET time_zone='Asia/Seoul'`이 `ERROR 1298`로 실패하는데, 이는 **`mysql.time_zone*` 테이블이 비어 있기 때문**입니다.
OS의 zoneinfo를 `mysql_tzinfo_to_sql`로 적재하면 해결됩니다.

---

## 2. 증상

### 2.1 CONVERT_TZ가 NULL
```sql
SELECT CONVERT_TZ('2021-09-02 15:18:00','UTC','Asia/Seoul');  -- NULL
-- 오프셋(+00:00, -09:00)은 동작하지만 '이름'은 NULL
SELECT CONVERT_TZ('2021-09-02 15:18:00','+00:00','-09:00');   -- 정상
```

### 2.2 time_zone 변경 실패
```sql
SET GLOBAL time_zone='Asia/Seoul';
-- ERROR 1298 (HY000): Unknown or incorrect time zone: 'Asia/Seoul'
```

---

## 3. 원인 — 타임존 테이블이 비어 있음

```sql
SELECT * FROM mysql.time_zone;       -- Empty set
SELECT * FROM mysql.time_zone_name;  -- Empty set
```

---

## 4. 조치 — zoneinfo 적재

```bash
# OS의 zoneinfo를 MySQL 타임존 테이블로 적재
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root -p mysql
```

> 적재 후 MySQL 재시작 또는 재접속.

---

## 5. 결과 확인

```sql
SELECT CONVERT_TZ('2021-09-02 15:18:00','UTC','Asia/Seoul');
-- 2021-09-03 00:18:00  (정상)

SELECT * FROM mysql.time_zone_name LIMIT 10;  -- 이름→id 매핑 채워짐
```

---

## 6. 연관 개념

- [[2026-06-13-21_(MySQL-Character-Set-설정)]]
- [[2026-06-13-43_(Percona-MySQL-운영-표준-my.cnf)]]
