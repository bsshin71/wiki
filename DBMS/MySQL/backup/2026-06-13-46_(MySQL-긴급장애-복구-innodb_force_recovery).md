# MySQL 긴급장애 복구 (innodb_force_recovery)

- **카테고리**: #DBMS #MySQL #Backup
- **태그**: #backup #장애복구 #innodb_force_recovery #InnoDB
- **작성일**: 2026-06-13
- **참조 원본**: [[2026-06-12-긴급장애 복구 - BigDataTeam]]

## 1. 핵심 요약

InnoDB 자동 복구가 불가능한 손상이 발생하면 MySQL은 종료됩니다. 이때 `innodb_force_recovery`(1~6)를 설정해 **강제 기동**한 뒤 `mysqldump`로 데이터를 빼내 재구축하는 것이 정석입니다.
값이 클수록 데이터 손실 위험이 커지므로 **가능한 한 낮은 값부터** 시도합니다.

---

## 2. 자동 복구 실패

- MySQL은 시작 시 항상 자동 복구를 수행한다.
- 자동으로 복구할 수 없는 손상이 있으면 복구를 중지하고 **종료**된다.

---

## 3. 적용 순서

| 상황 | 설정 |
|------|------|
| 로그 파일 손상 | `innodb_force_recovery=6` 으로 기동 |
| 데이터 파일 손상 | `innodb_force_recovery=1` 로 기동 |
| 원인 불확실 | `1 → 6` 순차로 올리며 기동 시도 |

> 값이 커질수록 심각한 상황 → **데이터 손실 가능성↑, 복구 가능성↓**.
> 강제 기동에 성공하면 즉시 `mysqldump`로 백업 후 DB를 재구축한다.

---

## 4. innodb_force_recovery 설정값

| 값 | 모드 | 설명 |
|----|------|------|
| 1 | SRV_FORCE_IGNORE_CORRUPT | 데이터/인덱스 페이지 손상("Database page corruption on disk")을 무시하고 기동. mysqldump·SELECT INTO OUTFILE로 덤프 후 재구축 권장 |
| 2 | SRV_FORCE_NO_BACKGROUND | 언두 레코드 삭제 과정 장애 시. 백그라운드 작업 중단 |
| 3 | SRV_FORCE_NO_TRX_UNDO | 미커밋 트랜잭션을 롤백하지 않고 기동. 기동 후 mysqldump로 백업·재구축 |
| 4 | SRV_FORCE_NO_IBUF_MERGE | 인서트(체인지) 버퍼 손상 시. 버퍼 내용 무시하고 강제 기동 |
| 5 | SRV_FORCE_NO_UNDO_LOG_SCAN | 언두 로그 사용 불가 시. 종료 시점 미커밋 작업도 커밋된 것처럼 처리(데이터 부정합) → 덤프 후 복구 |
| 6 | SRV_FORCE_NO_LOG_REDO | 리두 로그 손상 시. 리두 로그 전체 무시, **마지막 체크포인트 시점 데이터만** 남음. 리두 로그 삭제 후 기동 권장 → 덤프 후 복구 |

---

## 5. 복구 후 권장 절차

```sql
-- 강제 기동 성공 후
mysqldump -uroot -p --all-databases > rescue.sql
```

> 강제 기동 상태는 정상 운영 상태가 아니므로, 덤프로 데이터를 회수한 뒤
> 정상 초기화한 인스턴스에 재적재하여 DB를 재구축한다.

---

## 6. 연관 개념

- [[2026-06-13-28_(MySQL-Redo-Log-활성화-비활성화)]]
- [[2026-06-13-29_(MySQL-Undo-로그-관리)]]
- [[2026-06-13-44_(MySQL-시점-복구-PITR)]]
