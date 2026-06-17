# MySQL AWS RDS — general log·slow log 디스크 점유 해소

- **카테고리**: #DBMS #MySQL #admin
- **태그**: #MySQL #admin #AWS-RDS #general-log #slow-log #disk #monitoring
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-AWSRDSgeneral logslow log disk 점유관련 - BigDataTeam]]

## 1. 핵심 요약

AWS RDS MySQL에서 `general_log`·`general_log_backup` 테이블이 디스크를 과점유(프로비저닝 스토리지의 수 % 수준)하면 RDS 알림이 발생한다. `TRUNCATE TABLE`이 **불가**하므로 RDS 전용 프로시저 `mysql.rds_rotate_general_log` / `mysql.rds_rotate_slow_log`를 호출해야 한다.

---

## 2. 알림 예시

```
MySQL general and/or slow logs are consuming a large amount of provisioned storage.
generalLogSize: 119.36 MB, generalLogSizeBackup: 5.29 GB
```

## 3. 해소 방법

| 원인 | 조치 |
|------|------|
| `generalLogSize` 또는 `generalLogSizeBackup` 과점유 | `CALL mysql.rds_rotate_general_log;` |
| `binlogSize` 과점유 | `CALL mysql.rds_rotate_slow_log;` |

```sql
-- general log 정리 (1회 호출로 general_log → general_log_backup 이동 + general_log 초기화)
CALL mysql.rds_rotate_general_log;
-- general_log_backup까지 줄이려면 2회 호출
CALL mysql.rds_rotate_general_log;
CALL mysql.rds_rotate_general_log;

-- slow log 정리
CALL mysql.rds_rotate_slow_log;
```

> ⚠️ `TRUNCATE TABLE general_log`는 RDS에서 허용되지 않음. 반드시 rds_rotate 프로시저 사용.

## 4. 연관 개념

- [[2026-06-13-54_(MySQL-쿼리-로깅-general-slow-log)]] — general/slow log 설정
