# MySQL 데이터 이관 Shell 스크립트
- **카테고리**: #DBMS #MySQL
- **태그**: #MySQL #migration #데이터이관 #shell #mysqldump
- **작성일**: 2026-06-02
- **참조 원본**: [[2026-06-02-mysql데이터이관shellscript]]

## 1. 핵심 요약
- mysqldump를 활용한 MySQL 서버 간 테이블 단위 데이터 이관 Shell 스크립트. INSERT / REPLACE 두 가지 이관 모드 지원.

## 2. 상세 설명

### 스크립트 구성

```bash
# 소스/타겟 DB 설정
S_DATABASE=mydb           # 소스 DB명
T_DATABASE=mydb           # 타겟 DB명
S_HOST="server1"          # 소스 서버
S_PASSWD="****"
T_HOST="localhost"        # 타겟 서버
T_PASSWD="****"

# mysqldump 옵션
REPLACE_OPTION="--no-create-info --skip-add-drop-table --skip-add-locks --single-transaction --set-gtid-purged=OFF --replace"
CREATE_OPTION="--add-drop-table --set-gtid-purged=OFF"

# 실행 명령 정의
EXP_CMD_CRT="/usr/bin/mysqldump -h ${S_HOST} -uroot -p${S_PASSWD} $CREATE_OPTION --databases ${S_DATABASE}"
EXP_CMD_REP="/usr/bin/mysqldump -h ${S_HOST} -uroot -p${S_PASSWD} $REPLACE_OPTION --databases ${S_DATABASE}"
IMP_CMD="mysql -h ${T_HOST} -uroot -p${T_PASSWD} ${T_DATABASE} -A"

# 이관 대상 테이블 목록
TBLLIST="
T1
T2
"
```

### 이관 함수

**스키마만 이관** (데이터 없음):
```bash
function crtonly_runsql {
    for I in $TBLLIST; do
        $EXP_CMD_CRT_ONLY --tables $I > $I.dat
        $IMP_CMD < $I.dat
    done
}
```

**스키마 + 데이터 이관 (INSERT)**:
```bash
function crt_runsql {
    W=$1   # WHERE 조건 (선택)
    for I in $TBLLIST; do
        if [ "x${W}" != "x" ]; then
            $EXP_CMD_CRT --tables $I --where="${W}" > $I.dat
        else
            $EXP_CMD_CRT --tables $I > $I.dat
        fi
        $IMP_CMD < $I.dat
    done
}
```

**데이터 이관 (REPLACE — 중복 시 UPDATE)**:
```bash
function rep_runsql {
    W=$1
    for I in $TBLLIST; do
        $EXP_CMD_REP --tables $I > $I.dat
        # $IMP_CMD < $I.dat  # 필요시 주석 해제
    done
}
```

### 실행 예시

```bash
# 전체 테이블 스키마+데이터 이관 (조건 없음)
crt_runsql ""

# 조건부 이관
crt_runsql "created_at >= '2026-01-01'"
```

### 옵션 설명

| 옵션 | 설명 |
|------|------|
| `--no-create-info` | CREATE TABLE 문 제외 |
| `--skip-add-drop-table` | DROP TABLE 문 제외 |
| `--single-transaction` | InnoDB 일관성 보장 |
| `--set-gtid-purged=OFF` | GTID 복제 환경 대응 |
| `--replace` | INSERT 대신 REPLACE 사용 (중복 시 UPDATE) |

## 3. 연관 개념 (지식 연결)
_(관련 문서가 생성되면 여기에 백링크를 추가합니다.)_
