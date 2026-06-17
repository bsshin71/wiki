# MySQL Online DDL ALGORITHM (INSTANT·INPLACE·COPY)

- **카테고리**: #DBMS #MySQL #online-ddl
- **태그**: #MySQL #online-ddl #INSTANT #INPLACE #COPY #lock #알고리즘
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-대용량 테이블 온라인 DDL lock 최소로 잡기 - BigDataTeam]]

## 1. 핵심 요약

Online DDL 알고리즘은 잠금/부하가 작은 순으로 **INSTANT → INPLACE → COPY**. MySQL 8.0은 자동으로 INSTANT→INPLACE→COPY 순으로 가능한 것을 선택한다. **INSTANT(8.0.12+)** 는 메타데이터만 변경해 락이 거의 없어 최상이며, INPLACE(LOCK=NONE)는 시작·종료 시점에만 짧은 락, COPY는 임시테이블 복사로 DML 불가.

---

## 2. 알고리즘 비교

| 알고리즘 | 데이터 변경 | 락 | DML 중 |
|----------|-------------|-----|--------|
| **INSTANT** | 메타데이터만 | 거의 없음 | 가능(매우 짧은 대기) |
| **INPLACE** | 리빌드 가능(복사X) | 시작·종료 짧은 락 | 읽기/쓰기 가능 |
| **COPY** | 임시테이블 복사 후 RENAME | 큼 | 읽기만(DML 불가) |

## 3. INSTANT 사용·제약

```sql
ALTER TABLE foo ADD COLUMN bar1 INT, ADD COLUMN bar2 INT, ALGORITHM=INSTANT;
```
- 가능: 컬럼 추가, 기본값 추가/삭제, 인덱스 타입 변경, 테이블명 변경.
- 제약: **8.0.12+**, 컬럼 위치 지정 불가(마지막에만), ROW_FORMAT=COMPRESSED 불가, FULLTEXT/TEMPORARY 불가.

## 4. 서비스 영향 최소 시도 순서

```
1. ALGORITHM=INSTANT
2. ALGORITHM=INPLACE, LOCK=NONE
3. ALGORITHM=INPLACE, LOCK=SHARED
4. ALGORITHM=COPY, LOCK=SHARED
5. ALGORITHM=COPY, LOCK=EXCLUSIVE
```
> 1·2번이 안 되면 **DML을 멈추고** 변경해야 하는 작업.

## 5. row_log_buf · 진행률

```sql
SELECT EVENT_NAME, WORK_COMPLETED, WORK_ESTIMATED FROM performance_schema.events_stages_current;
-- row_log_buf 사용량
SELECT event_name, sys.format_bytes(current_alloc), sys.format_bytes(high_alloc)
FROM sys.x$memory_global_by_current_bytes WHERE event_name='memory/innodb/row_log_buf'\G;
SHOW GLOBAL VARIABLES LIKE '%online%';  -- innodb_online_alter_log_max_size 초과 시 실패
```

## 6. 연관 개념

- [[2026-06-14-MS11_(MySQL-대용량-테이블-DDL-주의사항-실패케이스)]]
- [[2026-06-14-MS13_(MySQL-대용량-Online-인덱스-추가-절차)]]
- [[2026-06-14-MS14_(MySQL-pt-online-schema-change-pt-osc)]]
