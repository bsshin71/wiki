# Percona MySQL 운영(PRD) 표준 my.cnf

- **카테고리**: #DBMS #MySQL #Installation
- **태그**: #install #my.cnf #파라미터 #운영설정
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-prd db용 my.cnf - BigDataTeam]]

## 1. 핵심 요약

운영(PRD) MySQL의 표준 my.cnf 설정값을 항목별로 정리한 레퍼런스입니다.
default와 동일한 값도 **관리 대상임을 명시하기 위해 의도적으로 유지**하며, PRD는 DEV와 달리 성능·안정성을 위해 일부 값을 조정합니다(예: `transaction-isolation=READ-COMMITTED`, `innodb_buffer_pool_size=8G`, `innodb_doublewrite=OFF`).

---

## 2. 기본 서버 설정

| 설정 | PRD 값 | Default | 설명 |
|------|--------|---------|------|
| server-id | N(정수) | 없음 | 복제 시 서버 고유 ID |
| datadir | /home/data/mysql | /var/lib/mysql/ | 데이터 파일 기본 경로 |
| socket | /var/lib/mysql/mysql.sock | /var/run/mysqld/mysqld.sock | 로컬 소켓 |
| pid-file | /home/data/trc/mysqld.pid | /var/run/mysqld/mysqld.pid | PID 파일 |

## 3. 스토리지 엔진

| 설정 | PRD 값 | Default | 설명 |
|------|--------|---------|------|
| default_storage_engine | InnoDB | InnoDB | 기본 엔진 |
| table_open_cache | 4000 | 2000 | 열린 테이블 캐시 |
| table_open_cache_instances | 16 | 16 | 캐시 분할(동시성) |

## 4. 파일·I/O

| 설정 | PRD 값 | Default | 설명 |
|------|--------|---------|------|
| open_files_limit | 65535 | OS 제한 | 열 수 있는 파일 수 |
| local_infile | OFF | OFF | LOAD DATA INFILE 허용 |
| tmpdir | /home/data/mytmp | /tmp | 임시 파일 경로 |

## 5. 트랜잭션·격리

| 설정 | PRD 값 | Default | 설명 |
|------|--------|---------|------|
| transaction-isolation | READ-COMMITTED | REPEATABLE-READ | 격리 수준 |
| autocommit | ON | OFF | 자동 커밋 |

## 6. SQL 모드·시간대

| 설정 | PRD 값 | Default |
|------|--------|---------|
| sql_mode | ONLY_FULL_GROUP_BY, STRICT_TRANS_TABLES, ERROR_FOR_DIVISION_BY_ZERO, NO_ENGINE_SUBSTITUTION | STRICT_TRANS_TABLES, NO_ENGINE_SUBSTITUTION |
| sysdate_is_now | ON | ON |
| default_time_zone | +00:00 | OS 기본값 |

## 7. 메모리·임시 테이블

| 설정 | PRD 값 | Default |
|------|--------|---------|
| max_heap_table_size | 16M | 16M |
| tmp_table_size | 16M | 16M |

## 8. 문자 집합

| 설정 | PRD 값 | Default |
|------|--------|---------|
| character_set_server | utf8mb4 | utf8mb4 |
| character_set_filesystem | utf8mb4 | binary |
| collation_server | utf8mb4_0900_ai_ci | utf8mb4_0900_ai_ci |
| skip-character-set-client-handshake | ON | N/A |

## 9. 연결·사용자

| 설정 | PRD 값 | Default | 설명 |
|------|--------|---------|------|
| max_connections | 600 | 151 | 동시 접속 수 |
| max_user_connections | 600 | 0(무제한) | 사용자별 접속 수 |
| max_connect_errors | 999999 | 100 | IP별 허용 오류 수 |
| skip_name_resolve | ON | OFF | 호스트명 미변환(성능) |
| connect_timeout | 20 | 10 | 연결 타임아웃 |
| interactive_timeout | 86400 | 28800 | 인터랙티브 종료 시간 |
| wait_timeout | 31536000 | 28800 | 비인터랙티브 종료 시간 |
| max_allowed_packet | 64M | 64M | 최대 패킷 크기 |

## 10. 바이너리 로그·복제

| 설정 | PRD 값 | Default | 설명 |
|------|--------|---------|------|
| log_bin | /home/data/log/mysql-bin/bin | OFF | binlog 활성화 |
| sync_binlog | 1 | 1 | binlog 동기화 빈도 |
| enforce_gtid_consistency | ON | OFF | GTID 일관성 강제 |
| gtid_mode | ON | OFF | GTID 사용 |
| binlog_format | ROW | ROW | binlog 형식 |
| max_binlog_size | 512M | 1G | binlog 파일 최대 크기 |
| relay_log_recovery | ON | OFF | 중계로그 복구 자동 |
| relay_log_purge | OFF/ON(상황) | ON | MHA(Slave>1)는 OFF, 1:1은 ON |

## 11. InnoDB

| 설정 | PRD 값 | Default | 설명 |
|------|--------|---------|------|
| innodb_sort_buffer_size | 64M | 1M | 정렬 버퍼 |
| innodb_data_home_dir | /home/data/mysql | ./ | 데이터 파일 위치 |
| innodb_log_file_size | 2048M | 48M | 로그 파일 크기 |
| innodb_log_files_in_group | 3 | 2 | 로그 파일 수 |
| innodb_undo_tablespaces | 2→8(대용량) | 2 | undo 테이블스페이스 |
| innodb_flush_log_at_trx_commit | 1 | 1 | 커밋 시 로그 플러시 |
| innodb_buffer_pool_size | 8G | 128M | 버퍼 풀 크기 |
| innodb_buffer_pool_instances | 8 | 8 | 버퍼 풀 인스턴스 |
| innodb_doublewrite | OFF | ON | SSD+백업 환경 성능 위해 OFF 가능 |

## 12. 로그

| 설정 | PRD 값 | Default |
|------|--------|---------|
| log_error | /home/data/trc/mysql-err.log | stderr |
| log_error_verbosity | 3 | 2 |
| slow_query_log | ON | OFF |
| long_query_time | 3 | 10 |

## 13. 병렬 복제(GTID)

| 설정 | PRD 값 | Default | 설명 |
|------|--------|---------|------|
| session_track_gtids | OWN_GTID | OFF | 세션 GTID 추적 |
| slave_parallel_type | LOGICAL_CLOCK | DATABASE | 멀티스레드 복제 방식 |
| slave_parallel_workers | 4 | 0 | 워커 수(CPU 50~70%) |
| slave_preserve_commit_order | 1 | 0 | 커밋 순서 보장(workers≥2) |

## 14. 보안·인증

| 설정 | PRD 값 | Default |
|------|--------|---------|
| validate_password_policy | LOW | MEDIUM |
| default_authentication_plugin | mysql_native_password | caching_sha2_password |
| read_only | 0 | OFF |

---

## 15. 연관 개념

- [[2026-06-13-14_(MySQL-Percona-설치-상세-가이드)]]
- [[2026-06-13-41_(Percona-MySQL-datadir-basedir-경로-변경)]]
- [[2026-06-13-26_(MySQL-Redo-Log-및-로그버퍼-설정)]]
