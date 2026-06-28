# Oracle PFILE과 SPFILE
- **카테고리**: #Oracle #admin
- **태그**: #Oracle #admin #PFILE #SPFILE #파라미터파일
- **작성일**: 2026-06-29
- **참조 원본**: [[2026-06-20_PFILE과SPILE]]
- **is_public**: true

## 목차
- [[#1. 핵심 요약]]
- [[#2. PFILE (정적 파라미터 파일)]]
- [[#3. SPFILE (동적 서버 파라미터 파일)]]
- [[#4. 비교표]]
- [[#5. SPFILE 파라미터 변경 명령어]]
- [[#6. 연관 개념]]

## 1. 핵심 요약
Oracle DB 기동 시 필요한 파라미터(SGA 크기, 최대 세션 수 등)를 저장하는 두 가지 파일. **PFILE**은 텍스트(편집 가능, 재기동 후 반영), **SPFILE**은 바이너리(SQL*Plus로만 수정, 즉시 반영 가능, RMAN 백업 지원). 현대 Oracle(9i 이후)은 SPFILE을 기본 사용하며, PFILE은 SPFILE 백업·비상 수정 용도로 보조 사용한다.

## 2. PFILE (정적 파라미터 파일)

- **파일명**: `init${ORACLE_SID}.ora` (예: `initdevdb.ora`)
- **형태**: 텍스트 — OS 에디터(`vi`, `notepad`)로 직접 편집 가능
- **반영**: DB **재기동 필수** (정적, 운영 중 변경 불가)
- **위치**: DB 서버 또는 클라이언트 PC에도 보관 가능

## 3. SPFILE (동적 서버 파라미터 파일)

- **파일명**: `spfile${ORACLE_SID}.ora` (예: `spfiledevdb.ora`)
- **형태**: 바이너리 — SQL*Plus `ALTER SYSTEM` 명령으로만 수정 (메모장 수정 시 파일 손상 → DB 기동 불가)
- **반영**: `SCOPE` 옵션에 따라 **즉시 반영** 또는 재기동 후 반영 선택 가능
- **위치**: DB 서버 전용. RMAN 자동 백업 지원

## 4. 비교표

| 구분 | PFILE | SPFILE |
|------|-------|--------|
| 파일 형태 | 텍스트 (Text) | 바이너리 (Binary) |
| 기본 파일명 | `init${ORACLE_SID}.ora` | `spfile${ORACLE_SID}.ora` |
| 수정 방법 | OS 에디터 (`vi`, `nano` 등) | SQL*Plus `ALTER SYSTEM` 명령 |
| 반영 시점 | **DB 재기동 필수** | `SCOPE`에 따라 즉시 또는 재기동 후 |
| RMAN 백업 | 불가 (수동 백업 필요) | 가능 (자동 백업 지원) |
| 도입 버전 | 전통 방식 | Oracle 9i 이후 (현재 기본값) |

## 5. SPFILE 파라미터 변경 명령어

```sql
-- 1. 메모리와 SPFILE 모두 즉시 변경 (현재 + 재기동 후 모두 적용)
ALTER SYSTEM SET open_cursors=300 SCOPE=BOTH;

-- 2. 현재 메모리에만 변경 (재기동하면 원래대로)
ALTER SYSTEM SET open_cursors=300 SCOPE=MEMORY;

-- 3. SPFILE에만 변경 (다음 재기동 시점부터 적용)
--    정적 파라미터(processes 등)는 반드시 SPFILE로만 변경 가능
ALTER SYSTEM SET processes=500 SCOPE=SPFILE;
```

**상호 변환:**
```sql
CREATE PFILE FROM SPFILE;   -- SPFILE → PFILE 백업
CREATE SPFILE FROM PFILE;   -- PFILE → SPFILE 복원
```

## 6. 연관 개념
- [[2026-06-29_(Oracle-Post-Installation-초기설정)]]
