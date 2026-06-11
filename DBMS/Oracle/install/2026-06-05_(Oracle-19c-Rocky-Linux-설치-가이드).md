# Oracle 19c Rocky Linux 설치 가이드
- **카테고리**: #DBMS #Oracle
- **태그**: #Oracle #install #Rocky-Linux #19c
- **작성일**: 2026-06-05
- **참조 원본**: [[2026-06-05-Rocky Linux 9.7 환경에 Oracle 19.3.0.0 설치과정]]

## 1. 핵심 요약
- Rocky Linux 9.x(GCC 11+)에서 Oracle 19c(19.3.0.0)를 설치하려면 **libpthread 심볼릭 링크 패치 + ins_rdbms.mk 수정 + 링크 에러 수동 relink** 3가지 우회 작업이 필수다.

## 2. 상세 설명

### Prerequisite — 필수 패키지 및 OS 권한 세팅 (root)

```bash
# 오라클 필수 패키지 + GUI 라이브러리
dnf install -y BC binutils compat-openssl11 elfutils-libelf elfutils-libelf-devel \
  fontconfig-devel glibc glibc-devel ksh libaio libaio-devel libXrender libXrender-devel \
  libX11 libX11-devel libXext libXext-devel libXtst libXtst-devel libgcc libstdc++ \
  libstdc++-devel libxcrypt-compat make sysstat targetcli smartmontools
dnf install -y xdpyinfo xorg-x11-apps

# ★ 핵심 패치: Rocky 9에서 폐기된 libpthread_nonshared.a 심볼릭 링크로 대체
rm -f /usr/lib64/libpthread_nonshared.a
ln -s /usr/lib64/libpthread.so.0 /usr/lib64/libpthread_nonshared.a
ldconfig

# GUI 화면 소유권 허용 (데스크톱 터미널에서 실행)
xhost +
```

---

### Step 1 — 환경변수 설정 및 압축 해제 (oracle 계정)

`~/.bash_profile`에 아래 내용을 추가 후 `source ~/.bash_profile` 적용.

```bash
export ORACLE_BASE=/oracle/u01/app/oracle
export ORACLE_HOME=$ORACLE_BASE/product/19.3.0/dbhome_1
export ORA_INVENTORY=/oracle/u01/app/oraInventory
export ORACLE_SID=devdb
export TNS_ADMIN=$ORACLE_HOME/network/admin
export NLS_LANG=AMERICAN_AMERICA.AL32UTF8
export PATH=$ORACLE_HOME/bin:$ORACLE_HOME/OPatch:$PATH
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib
export CLASSPATH=$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
export CV_ASSUME_DISTID=RHEL9   # Rocky를 RHEL로 인식시키는 핵심 변수
export LANG=C
export DISPLAY=:0.0
```

```bash
mkdir -p $ORACLE_HOME
cd $ORACLE_HOME
unzip -q /home/oracle/install/V982063-01.zip
```

---

### Step 2 — 설치 전 선제 파일 수정 (oracle 계정)

GCC 11+의 엄격한 링커 에러를 방어하기 위해 메이크파일을 미리 패치한다.

```bash
cd $ORACLE_HOME/rdbms/lib
sed -i 's/\$(DL_OPTS)/\$(DL_OPTS) -Wl,--no-as-needed/g' ins_rdbms.mk
```

---

### Step 3 — GUI 설치 실행

```bash
cd $ORACLE_HOME
./runInstaller
```

**마법사 선택 가이드:**

| 화면 | 선택 |
|------|------|
| Configure Option | `Set up Software Only` |
| Database Installation Options | `Single instance database installation` |
| Database Edition | `Enterprise Edition` |
| Installation Locations | 환경변수 경로 확인 (`/oracle/u01/app/oracle`) |
| Operating System Groups | 기본값 유지 |
| **Prerequisite Checks** | ⚠️ **`Ignore All` 체크 후 Next** |

---

### Step 4 — ★ 링크 에러 발생 시 수동 패치 (가장 중요)

설치 게이지 **11~70% 구간**에서 아래 에러가 발생하며 대기 상태에 걸린다:
> *Error in invoking target 'libasmclntsh19.ohso libasmperl19.ohso client_sharedlib' of makefile...*

> ⚠️ **팝업창을 건들지 말 것. Abort/Retry/Continue 아무것도 누르지 않는다.**

새 터미널을 열어 `oracle` 계정으로 아래 명령어를 순서대로 실행:

```bash
# 1. 구형 stub 파일 숨기기
cd $ORACLE_HOME/lib/stubs
mv libc.so libc.so.bak

# 2. 핵심 클라이언트 공유 라이브러리 수동 선행 컴파일
cd $ORACLE_HOME/network/lib
export LDFLAGS="-Wl,--copy-dt-needed-entries -Wl,--no-as-needed"
make -f ins_net_client.mk client_sharedlib

# 3. GCC 11 완화 플래그로 전체 relink
cd $ORACLE_HOME/bin
export CFLAGS="-O2 -fpermissive -Wno-error=implicit-function-declaration"
export CXXFLAGS="-O2 -fpermissive -Wno-error=implicit-function-declaration"
export LDFLAGS="-Wl,--copy-dt-needed-entries -Wl,--no-as-needed"
./relink all
```

`relink all`이 에러 없이 완료되면 → GUI 팝업의 **`[Retry]`** 클릭 → 게이지 100% 진행.

---

### Step 5 — 포스트 권한 스크립트 (root)

```bash
/oracle/u01/app/oraInventory/orainstRoot.sh
/oracle/u01/app/oracle/product/19.3.0/dbhome_1/root.sh
```

---

### 핵심 우회 기법 정리

| 문제 | 원인 | 해결책 |
|------|------|--------|
| `libpthread_nonshared.a` 없음 | Rocky 9에서 폐기 | 심볼릭 링크로 대체 |
| Prerequisite 패키지 경고 | 구형 compat 패키지 부재 | Ignore All 체크 |
| Link binaries 에러 | GCC 11+ 엄격한 링커 | `ins_rdbms.mk` 패치 + 수동 `relink all` |

## 3. 연관 개념 (지식 연결)
- [[2026-06-05_(Oracle-유저-생성하기)]] — 설치 완료 후 사용자 계정 생성 다음 단계
- [[2026-06-05_(Oracle-리스너-등록)]] — 설치 완료 후 리스너 등록 설정
