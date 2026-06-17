# MySQL Definer vs Invoker (SQL SECURITY)

- **카테고리**: #DBMS #MySQL #admin
- **태그**: #MySQL #admin #definer #invoker #stored-procedure #SQL-SECURITY #권한
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-MySQL Definer vs Invoker - BigDataTeam]]

## 1. 핵심 요약

프로시저·함수·뷰의 **SQL SECURITY 속성**이 `DEFINER`(기본)이면 **생성자 권한**으로 실행되고, `INVOKER`이면 **호출자 권한**으로 실행된다. 트리거·이벤트는 SQL SECURITY가 없고 항상 DEFINER 컨텍스트. DEFINER 계정이 삭제·변경되면 DEFINER 방식은 문제가 생길 수 있다.

---

## 2. 핵심 차이

| 항목 | SQL SECURITY DEFINER (기본) | SQL SECURITY INVOKER |
|------|---------------------------|---------------------|
| 실행 권한 | 생성자(DEFINER) 권한 | 호출자(INVOKER) 권한 |
| 보안성 | 직접 테이블 접근 차단 가능 (더 안전) | 호출자 권한 그대로 (더 투명) |
| 감사 로그 | 실행 사용자 불명확할 수 있음 | 호출자 기록 명확 |
| 유지보수 | DEFINER 계정 변경 시 위험 | DEFINER 계정과 무관 |

## 3. 예시

```sql
-- DEFINER 방식: definer_user 권한으로 실행
CREATE DEFINER = 'definer_user'@'%' PROCEDURE p1()
SQL SECURITY DEFINER
BEGIN
  UPDATE t1 SET counter = counter + 1;  -- definer_user에 UPDATE 권한 필요
END;

-- INVOKER 방식: 호출자 권한으로 실행
CREATE DEFINER = 'definer_user'@'%' PROCEDURE p2()
SQL SECURITY INVOKER
BEGIN
  UPDATE t1 SET counter = counter + 1;  -- 호출자에 UPDATE 권한 필요
END;
```

## 4. 언제 사용?

| 상황 | 추천 |
|------|------|
| 민감 데이터 숨기고 프로시저로만 접근 허용 | DEFINER |
| 보고서 생성·DBA 중앙 관리 | DEFINER |
| 멀티 테넌트·Row-level 접근제어 | INVOKER |
| 감사 로그·DEFINER 변경 가능성 높음 | INVOKER |

## 5. DEFINER 정보 조회

```sql
SELECT routine_schema, routine_name, routine_type, definer
FROM INFORMATION_SCHEMA.ROUTINES
WHERE routine_schema = 'BOSRL';
```

## 6. 연관 개념

- [[2026-06-13-47_(MySQL-User-계정-생성-옵션)]]
- [[2026-06-13-48_(MySQL-권한-관리-체계)]]
