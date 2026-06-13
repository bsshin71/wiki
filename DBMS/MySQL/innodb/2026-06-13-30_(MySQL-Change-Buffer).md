# MySQL Change Buffer

- **카테고리**: #DBMS #MySQL #InnoDB
- **태그**: #InnoDB #change_buffer #인덱스 #버퍼링
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-체인지버퍼 - BigDataTeam]]

## 1. 핵심 요약

Change Buffer는 **세컨더리 인덱스 변경을 즉시 디스크에 반영하지 않고 메모리에 버퍼링**했다가 나중에 일괄 반영하는 임시 공간입니다.
디스크 random I/O를 줄여 쓰기 성능을 높이며, `innodb_change_buffering`으로 버퍼링 대상 작업을 제어합니다.

---

## 2. 개념

인덱스 변경 시 데이터 파일을 즉시 변경하는 작업은 자원 소모가 큽니다.
이를 즉시 반영하지 않고 변경 내역을 버퍼링해 나중에 반영하는데, 그 변경 내역을 저장하는 임시 메모리 공간이 **Change Buffer**입니다.

---

## 3. 관련 프로퍼티

```sql
SHOW VARIABLES LIKE 'innodb_change%';
-- innodb_change_buffer_max_size | 25       (버퍼풀 대비 최대 %)
-- innodb_change_buffering       | inserts
```

### innodb_change_buffering 설정값

| 설정값 | 설명 |
|--------|------|
| `all` | 모든 인덱스 작업(inserts, deletes, purges) 버퍼링 |
| `none` | 버퍼링 안 함 |
| `inserts` | 인덱스 추가 작업만 버퍼링 |
| `deletes` | 인덱스 삭제 작업만 버퍼링 |
| `changes` | 추가 + 삭제(inserts + deletes) 버퍼링 |
| `purges` | 영구 삭제(백그라운드 purge) 작업만 버퍼링 |

---

## 4. 사용량 조회

```sql
-- Change Buffer 메모리 사용량
SELECT event_name, current_number_of_bytes_used
FROM performance_schema.memory_summary_global_by_event_name
WHERE event_name='memory/innodb/ibuf0ibuf';
-- memory/innodb/ibuf0ibuf | 144
```

```sql
-- INNODB STATUS 의 INSERT BUFFER 섹션
SHOW ENGINE INNODB STATUS \G;
-- INSERT BUFFER AND ADAPTIVE HASH INDEX
-- Ibuf: size 1, free list len 0, seg size 2, 0 merges
```

---

## 5. 연관 개념

- [[2026-06-13-26_(MySQL-Redo-Log-및-로그버퍼-설정)]]
- [[2026-06-13-25_(MySQL-Index-최적화)]]
- [[2026-06-13_(MySQL-Performance-Schema-활용)]]
