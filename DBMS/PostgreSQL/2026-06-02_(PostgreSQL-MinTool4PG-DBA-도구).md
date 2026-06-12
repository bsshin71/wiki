# PostgreSQL MinTool4PG (DbToys4PG) DBA 도구
- **카테고리**: #DBMS #PostgreSQL
- **태그**: #PostgreSQL #DBA #Windows #도구 #tool
- **작성일**: 2026-06-02
- **참조 원본**: raw/2026-06-01-DbToys4Pg설치법.txt

## 1. 핵심 요약
- Windows Terminal 환경에서 PostgreSQL DBA 작업을 지원하는 Windows용 스크립트 도구. WSL2/PowerShell이 막힌 프로젝트 환경 대응용.

## 2. 상세 설명

### 대상 환경
- WSL2 또는 PowerShell이 막혀 있고 Windows Terminal만 가능한 환경
- 기존 Linux 터미널 스크립트를 Windows에서 사용하기 위해 작성

### 주요 기능
- 테이블 정보: 인덱스, 통계정보 조회 (DB 엔진 기본 기능 활용)
- 가독성 높은 PLAN 출력
- Active Session / Lock Wait 모니터링
- DB Properties 비교
- DDL 추출 (`DBMS_METADATA.GET_DDL` 유사)

### 설치

**사전 요구사항**:
1. **Windows용 GIT 설치** → MinGW 포함 (`cat`, `grep`, `sed`, `find`, `awk` 등 사용 가능)
2. **Windows용 PostgreSQL CLI (psql) 설치**
   - 경로: `C:\Program Files\PostgreSQL\17\bin`
3. `MinTool4PG-lastest.zip`을 `%USERPROFILE%\Desktop\DBA`에 압축 해제

**환경변수 설정** ("사용자 환경 변수"):
```
DBAHOME = %USERPROFILE%\Desktop\DBA
MINGW   = C:\Program Files\Git\usr\bin
```

**PATH 추가**:
```
C:\Program Files\PostgreSQL\17\bin
%MINGW%
%DBAHOME%\bin
```

---

### pgpass.conf 파일 설정

PostgreSQL 비밀번호 자동 인증을 위한 설정 파일.

**위치 설정** (시스템 환경 변수):
```
PGPASSFILE = %DBAHOME%\bin\config\pgpass.conf
```

**파일 내용 형식** (`hostname:port:database:username:password`):
```
127.0.0.1:5432:*:postgres:passwd
192.168.1.50:5432:test_db:dev_user:dev_pass
```

> 💡 파일 탐색기 주소창에 `%APPDATA%` 입력 후 `postgresql` 폴더 생성하여 파일 생성 가능.

## 3. 연관 개념 (지식 연결)
- [[2026-06-02_(PostgreSQL-RPM-설치-가이드)]] — PostgreSQL 서버 설치
- [[2026-06-02_(PostgreSQL-운영-유틸리티-모음)]] — psql 명령어 및 운영 도구
