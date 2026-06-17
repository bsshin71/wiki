# MySQL 엔진 잠금 (Global·Table·Named·Metadata)

- **카테고리**: #DBMS #MySQL #lock
- **태그**: #MySQL #lock #global-lock #table-lock #named-lock #metadata-lock
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-MySQL 엔진잠금 - BigDataTeam]]

## 1. 핵심 요약

MySQL 서버 레벨(엔진) 잠금은 범위가 큰 순으로 **글로벌 → 테이블 → 네임드 → 메타데이터** 락이 있다. 글로벌락은 서버 전체(백업용), 테이블락은 개별 테이블(InnoDB는 DML 무시·DDL만 영향), 네임드락은 문자열 대상(앱 동기화), 메타데이터락은 객체 이름/구조 변경 시 자동 획득된다.

---

## 2. 잠금별 요약

| 잠금 | 범위 | 획득 | 사용 예 |
|------|------|------|---------|
| **글로벌락** | 서버 전체 | `FLUSH TABLES WITH READ LOCK` | mysqldump 일관 백업. **8.0+ `LOCK INSTANCE FOR BACKUP`** 권장 |
| **테이블락** | 개별 테이블 | `LOCK TABLES t [READ|WRITE]` / `UNLOCK TABLES` | InnoDB는 변경 DML 무시·DDL만 영향. 독점 테이블 변경 |
| **네임드락** | 사용자 문자열 | `GET_LOCK('s',n)` / `IS_FREE_LOCK` / `RELEASE_LOCK` | 서버간 동기화, 배치 실행순서 분산 |
| **메타데이터락(MDL)** | 객체 이름/구조 | 자동(`RENAME TABLE` 등) | `RENAME TABLE rank TO rank_backup, rank_new TO rank` (원본+대상 동시 잠금) |

> 글로벌락 획득 시 다른 세션은 SELECT 제외 대부분의 DDL/DML이 대기.
> MDL은 명시적 획득/해제 불가, 원본과 변경 대상 2개를 한꺼번에 잠금.

## 3. 연관 개념

- [[2026-06-14-MS03_(MySQL-InnoDB-Locking-Transaction-Model)]] — InnoDB 레벨 락과 함께 보기
- [[2026-06-13-17_(MySQL-Meta-Lock-세션-조회)]]
