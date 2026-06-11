# Oracle DBCA & NETCA 가이드
- **카테고리**: #DBMS #Oracle
- **태그**: #Oracle #install #DBCA #NETCA
- **작성일**: 2026-06-05
- **참조 원본**: [[2026-06-05-DBCA 와 NETCA 실행방법]]

## 1. 핵심 요약
- Oracle 19c에서 DB 인스턴스 생성은 `dbca -silent`, 리스너 구성은 `netca -silent`로 무인(Silent) 실행한다.
- `dbca.rsp` 응답 파일은 CamelCase 및 변수 순서를 엄격히 지켜야 파싱 오류가 없다.

## 2. 상세 설명

### 사전 작업 — 데이터 디렉터리 생성 (oracle 계정)

```bash
mkdir -p /oracle/u01/app/oracle/oradata
```

---

### Step 1 — dbca.rsp 응답 파일 작성

```bash
cat << 'EOF' > /home/oracle/dbca.rsp
createDatabase=true
gdbName=devdb
sid=devdb
databaseConfigType=SI
templateName=General_Purpose.dbc
sysPassword=Oracle19c!
systemPassword=Oracle19c!
emConfiguration=NONE
dbsnmpPassword=Oracle19c!
characterSet=AL32UTF8
nationalCharacterSet=AL16UTF16
storageType=FS
datafileDestination=/oracle/u01/app/oracle/oradata
sampleSchema=false
automaticMemoryManagement=false
totalMemory=0
EOF
```

> ⚠️ 키 이름은 CamelCase, 섹션 구분자(`[...]`) 없이 작성해야 한다.

---

### Step 2 — DB 인스턴스 무인 생성

```bash
source ~/.bash_profile
dbca -silent -createDatabase -responseFile /home/oracle/dbca.rsp
```

- 40~50% 구간에서 내부 딕셔너리 뷰 컴파일로 **10~20분 대기는 정상**이다.
- `100% 완료` + `데이터베이스 생성이 완료되었습니다.` 메시지 확인.

---

### Step 3 — NETCA로 리스너 무인 구성

```bash
netca -silent -responseFile $ORACLE_HOME/assistants/netca/netca.rsp
lsnrctl status
```

- `lsnrctl status` 하단에 `Instance "devdb", status READY` 가 보이면 성공.

---

### Step 4 — DB 상태 최종 검증

```sql
sqlplus / as sysdba

SELECT name, open_mode, database_role FROM v$database;
-- OPEN_MODE = READ WRITE 확인

SELECT * FROM nls_database_parameters WHERE parameter='NLS_CHARACTERSET';
-- VALUE = AL32UTF8 확인

exit;
```

---

### Step 5 — 리눅스 재부팅 시 자동 시작 설정 (운영 필수)

**`/etc/oratab` 수정**:
```
# 변경 전
devdb:/oracle/u01/app/oracle/product/19.3.0/dbhome_1:N
# 변경 후
devdb:/oracle/u01/app/oracle/product/19.3.0/dbhome_1:Y
```

**수동 제어 명령어**:
```bash
dbshut $ORACLE_HOME    # DB + 리스너 일괄 종료
dbstart $ORACLE_HOME   # DB + 리스너 일괄 시작
```

---

### 단계별 흐름 요약

```
엔진 설치 → dbca.rsp 작성 → dbca -silent → netca -silent → lsnrctl status → v$database 검증 → oratab 자동시작 등록
```

## 3. 연관 개념 (지식 연결)
- [[2026-06-05_(Oracle-19c-Rocky-Linux-설치-가이드)]] — DB 인스턴스 생성 전 엔진 설치 단계
- [[2026-06-05_(Oracle-리스너-등록)]] — 리스너 동적·정적 등록 상세
- [[2026-06-05_(Oracle-유저-생성하기)]] — DB 생성 후 사용자 계정 생성 단계
