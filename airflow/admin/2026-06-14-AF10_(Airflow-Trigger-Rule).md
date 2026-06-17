# Airflow Trigger Rule (트리거 규칙)

- **카테고리**: #airflow #admin
- **태그**: #airflow #admin #trigger_rule #dag #dependency
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-airflow trigger rule - BigDataTeam]]

## 1. 핵심 요약

Trigger Rule은 **상위(업스트림) 태스크 상태에 따라 해당 태스크의 실행 조건**을 정의한다. 기본값은 `all_success`(모든 부모 성공). 오류 처리·알림·정리 작업에는 `one_failed`/`all_done` 등을 사용한다.

---

## 2. 지원 트리거 규칙

| 트리거 규칙 | 동작 | 사용 사례 |
|-------------|------|-----------|
| **all_success** (기본) | 모든 상위 태스크 성공 시 트리거 | 일반 워크플로우 |
| all_failed | 모든 상위 태스크 실패/오류 시 | 다수 실패 예상 시 오류 처리 |
| all_done | 결과 무관, 모든 부모 완료 시 | 시스템 종료·정리(clean up) |
| one_failed | 하나 이상 실패하자마자(대기 안 함) | 알림·롤백 빠른 트리거 |
| one_success | 한 부모 성공하자마자(대기 안 함) | 결과 가용 즉시 다운스트림 실행 |
| none_failed | 실패한 부모 없음(성공/건너뜀 허용) | 조건부 브랜치 결합 |
| none_skipped | 건너뛴 부모 없음(성공/실패 허용) | 모든 업스트림 실행됨 |
| dummy | 부모 상태 무관 트리거 | 테스트 |

## 3. 활용 패턴

```python
send_success = TelegramOperator(..., trigger_rule="all_success")
send_failure = TelegramOperator(..., trigger_rule="one_failed")
backup_task >> [send_success, send_failure]
```

## 4. 연관 개념

- [[2026-06-14-AF04_(Airflow-기본-Usecase-SSHOperator-Telegram)]] — all_success/one_failed 알림
- [[2026-06-14-AF11_(Airflow-파티션-완료확인-알림-DAG)]]
