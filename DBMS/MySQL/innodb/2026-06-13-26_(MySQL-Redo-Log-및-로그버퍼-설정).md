# MySQL Redo Log 및 로그버퍼 설정

- **카테고리**: #DBMS #MySQL #InnoDB
- **태그**: #InnoDB #redo_log #로그버퍼 #튜닝
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-리두로그 및 로그버퍼 - BigDataTeam]]

## 1. 핵심 요약

Redo Log는 InnoDB의 변경 사항을 디스크에 기록하여 장애 시 복구를 보장합니다.
`innodb_flush_log_at_trx_commit`(0/1/2)로 내구성과 성능을 조절하며, `innodb_log_buffer_size`로 로그 버퍼 크기를 관리합니다.

---

## 2. innodb_flush_log_at_trx_commit (내구성 vs 성능)

| 설정값 | 동작 |
|--------|------|
| **0** | 1초에 한 번씩 redo log를 디스크로 write·동기화. 서버 비정상 종료 시 일부 데이터 유실 가능 |
| **1** | 매 트랜잭션 커밋마다 디스크 기록·동기화 (**기본값**, 가장 안전) |
| **2** | 매 커밋마다 디스크로 write 하지만 sync(동기화)는 1초에 한 번 |

> 대량 적재·로그성 데이터는 0 또는 2로 성능 향상 가능. 금융·결제성은 1 유지.

---

## 3. 주요 파라미터

| 파라미터 | 설명 |
|----------|------|
| `innodb_log_file_size` | redo log 파일 크기 (예: 1073741824 = 1GB) |
| `innodb_log_files_in_group` | redo log 파일 개수 |
| `innodb_log_buffer_size` | 로그 버퍼 크기. 기본 16M. BLOB/TEXT 자주 쓰면 증설 |

---

## 4. 관련 프로퍼티 조회

```sql
SHOW VARIABLES LIKE 'innodb_flush%';
-- innodb_flush_log_at_trx_commit | 1
-- innodb_flush_method            | O_DIRECT
-- innodb_flush_neighbors         | 0

SHOW VARIABLES LIKE 'innodb_log%';
-- innodb_log_buffer_size  | 16777216   (16M)
-- innodb_log_file_size    | 1073741824 (1GB)
-- innodb_log_files_in_group | 2
-- innodb_log_writer_threads | ON
```

---

## 5. 연관 개념

- [[2026-06-13-27_(MySQL-Redo-Log-아카이빙)]]
- [[2026-06-13-28_(MySQL-Redo-Log-활성화-비활성화)]]
- [[2026-06-13-29_(MySQL-Undo-로그-관리)]]
- [[2026-06-13-30_(MySQL-Change-Buffer)]]
