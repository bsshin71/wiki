# Oracle 19c EE 설치 가이드 (RedHat7, Non-CDB)
- **카테고리**: #Oracle #install
- **태그**: #Oracle #install #19c #EE #RedHat7 #Non-CDB
- **작성일**: 2026-06-29
- **참조 원본**: [[2026-06-21_oracle 19c EE 설치 on redhat7]]
- **is_public**: true

## 목차
- [[#1. 핵심 요약]]
- [[#2. 설치 절차]]
- [[#3. 연관 개념]]

## 1. 핵심 요약
RedHat Enterprise 7에 Oracle 19c Enterprise Edition을 RPM 방식으로 Non-CDB 설치하는 절차. preinstall 패키지 → Oracle binary → DBCA Non-CDB 생성 → 환경변수 설정 → 리스너 설정 순서로 진행. Rocky Linux 설치(GCC 11+ 충돌 이슈)와 달리 RHEL7은 preinstall 패키지 설치 후 바로 RPM 설치가 가능하다.

## 2. 설치 절차

### Step 1. 설치 파일 준비
Oracle 19c EE Linux RPM을 개인 PC에 다운로드 후 서버에 전송(WinSCP 등).

### Step 2. preinstall 패키지 설치 (root)
```bash
# 2-1. preinstall 패키지 다운로드
curl -o oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm \
  https://yum.oracle.com/repo/OracleLinux/OL7/latest/x86_64/getPackage/oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm

# 2-2. 설치
yum -y localinstall oracle-database-preinstall-19c-1.0-1.el7.x86_64.rpm
```

### Step 3. Oracle 19c EE RPM 설치 (root)
```bash
yum -y localinstall oracle-database-ee-19c-1.0-1.x86_64.rpm
```

### Step 4. oracle 유저 환경변수 설정
```bash
# .bashrc에 추가 후 . .bashrc 적용
export ORACLE_BASE=/opt/oracle
export ORACLE_HOME=/opt/oracle/product/19c/dbhome_1
export PATH=$PATH:$ORACLE_HOME/bin
```

### Step 5. ORACLE_SID 설정
```bash
export ORACLE_SID=ORCL
```

### Step 6. Non-CDB DB 생성 (DBCA)
```bash
dbca -silent -createDatabase \
  -templateName General_Purpose.dbc \
  -gdbName ORCL -sid ORCL \
  -sysPassword welcome1 -systemPassword welcome1 \
  -emConfiguration NONE \
  -datafileDestination /opt/oracle/oradata \
  -storageType FS \
  -characterSet AL32UTF8
```

### Step 7. 리스너 설정 (선택)

`$ORACLE_HOME/network/admin/listener.ora` 설정:

```
LISTENER =
  (ADDRESS_LIST=
    (ADDRESS=(PROTOCOL=tcp)(HOST=orcl05)(PORT=1521))
    (ADDRESS=(PROTOCOL=ipc)(KEY=PNPKEY)))

SID_LIST_LISTENER=
  (SID_LIST=
    (SID_DESC=
      (GLOBAL_DBNAME=ORCL)
      (SID_NAME=ORCL)
      (ORACLE_HOME=/opt/oracle/product/19c/dbhome_1)))
```

### Step 8. 리스너 시작
```bash
lsnrctl start
```

## 3. 연관 개념
- [[2026-06-05_(Oracle-19c-Rocky-Linux-설치-가이드)]]
- [[2026-06-05_(Oracle-리스너-등록)]]
- [[2026-06-05_(Oracle-DBCA-NETCA-가이드)]]
- [[2026-06-05_(Oracle-유저-생성하기)]]
