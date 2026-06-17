# MySQL MTS(Multi-Threaded Slave) 통계 로그 끄기

- **카테고리**: #DBMS #MySQL #replication
- **태그**: #MySQL #replication #MTS #slave #log_error_verbosity #troubleshooting
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-Multi-Threaded Slave Statistics  로깅 끄기 - BigDataTeam]]

## 1. 핵심 요약

`slave_parallel_workers > 1`(MTS 병렬 복제 활성) 시 mysqld.log에 **120초마다 MTS 통계 NOTE 로그**가 과도하게 찍힌다. NOTE 단계 로그를 모두 끄려면 `log_error_verbosity=2`로 올리면 되나, NOTE가 전부 제외되는 부작용이 있다.

---

## 2. 현상

```sql
SHOW VARIABLES LIKE 'slave_parallel%';
-- slave_parallel_workers: 4, slave_parallel_type: LOGICAL_CLOCK

-- mysqld.log에 120초마다:
-- 2021-11-18T01:05:09 ... Multi-threaded slave statistics for channel '':
-- seconds elapsed=121; events assigned=11592705; ...
```

## 3. 조치

```sql
SET GLOBAL log_error_verbosity = 2;
-- 기본값 3(INFO+WARNING+ERROR) → 2(WARNING+ERROR)로 올려 NOTE 제외
```

> ⚠️ NOTE 로그 전체가 제외되는 부작용. 경고·에러만 봐도 충분한 환경에서 적용.

## 4. 연관 개념

- [[2026-06-14-MS21_(MySQL-이중화-지연-원인분석-MTS-병렬복제-설정)]] — MTS 설정 활용
- [[2026-06-13-02_(MySQL-GTID-기반-복제)]]
