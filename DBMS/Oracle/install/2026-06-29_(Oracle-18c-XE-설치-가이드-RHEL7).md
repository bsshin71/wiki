# Oracle 18c XE 설치 가이드 (RHEL7, Non-CDB)
- **카테고리**: #Oracle #install
- **태그**: #Oracle #install #18c #XE #RHEL7 #Non-CDB
- **작성일**: 2026-06-29
- **참조 원본**: [[2026-06-20_oracle 18c XE install]]
- **is_public**: true

## 목차
- [[#1. 핵심 요약]]
- [[#2. 사전 확인]]
- [[#3. 설치 절차]]
- [[#4. 연관 개념]]

## 1. 핵심 요약
RedHat Enterprise 7에 Oracle 18c XE를 Non-CDB(PDB 없이) 방식으로 설치하는 가이드. RPM 방식으로 설치 후 `/etc/init.d/oracle-xe-18c` 스크립트의 PDB 관련 환경 변수를 주석 처리하여 단순 Non-CDB DB를 구성한다. 기본적으로 CDB/PDB 기반으로 생성되므로 수동 수정이 필요하다.

## 2. 사전 확인
- OS: RedHat Enterprise Linux 7
- `wget` 설치 필요: `sudo yum install wget`

## 3. 설치 절차

### Step 1. 18c XE RPM 다운로드
```bash
wget https://download.oracle.com/otn-pub/otn_software/db-express/oracle-database-xe-18c-1.0-1.x86_64.rpm
```

### Step 2. preinstall 패키지 설치 (root)
```bash
curl -o oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm \
  https://yum.oracle.com/repo/OracleLinux/OL7/latest/x86_64/getPackage/oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm
yum -y localinstall oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm
rm oracle-database-preinstall-18c-1.0-1.el7.x86_64.rpm
```

### Step 3. Oracle 바이너리 설치 (root)
```bash
yum -y localinstall oracle-database-xe-18c-1.0-1.x86_64.rpm
```

### Step 4. Non-CDB DB 생성을 위한 스크립트 수정

기본 설치 스크립트는 CDB/PDB 기반으로 동작. 이를 Non-CDB로 변경한다.

**4-1.** `/etc/init.d/oracle-xe-18c` — PDB 환경변수 주석 처리
```bash
#export PDB_NAME=XEPDB1
#export NUMBER_OF_PDBS=1
#export CREATE_AS_CDB=true
```

**4-2.** 동일 파일의 `$DBCA` 명령에서 `-createAsCDB` 옵션 제거 (Non-CDB 형태)
```bash
$SU -s /bin/bash $ORACLE_OWNER -c "(echo '$ORACLE_PASSWORD'; echo '$ORACLE_PASSWORD'; echo '$ORACLE_PASSWORD') | \
  $DBCA -silent -createDatabase -gdbName $ORACLE_SID -templateName $TEMPLATE_NAME \
  -characterSet $CHARSET -sid $ORACLE_SID -emConfiguration DBEXPRESS \
  -emExpressPort $EM_EXPRESS_PORT \
  -J-Doracle.assistants.dbca.validate.DBCredentials=false \
  -sampleSchema true $SQLSCRIPT_CONSTRUCT $DBFILE_CONSTRUCT $MEMORY_CONSTRUCT"
```

**4-3.** `/opt/oracle/product/18c/dbhomeXE/assistants/dbca/postdb_creation.sql` — PDB 구문 주석 처리
```sql
--ALTER PLUGGABLE DATABASE XEPDB1 SAVE STATE;
--alter session set container=XEPDB1;
```

**4-4.** Non-CDB로 DB 생성 실행 (root)
```bash
/etc/init.d/oracle-xe-18c configure
```

### Step 5. oracle 유저 환경변수 설정
```bash
# su - oracle 후 .bashrc에 추가
export ORACLE_BASE=/opt/oracle
export ORACLE_HOME=/opt/oracle/product/18c/dbhomeXE
export ORACLE_SID=ORCLXE
export PATH=$PATH:$ORACLE_HOME/bin
```

## 4. 연관 개념
- [[2026-06-05_(Oracle-19c-Rocky-Linux-설치-가이드)]]
- [[2026-06-05_(Oracle-DBCA-NETCA-가이드)]]
- [[2026-06-05_(Oracle-리스너-등록)]]
