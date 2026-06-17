# Airflow 설치 (pip · Metadb 설정)

- **카테고리**: #airflow #install
- **태그**: #airflow #install #pip #metadb #scheduler
- **작성일**: 2026-06-14
- **참조 원본**: [[2026-06-14-airflow 설치 - BigDataTeam]]

## 1. 핵심 요약

Python3 venv에 `pip install apache-airflow`로 설치하고, Metadb를 SQLite(테스트) 또는 **PostgreSQL(권장, LocalExecutor)** 로 설정한 뒤 `airflow db init` → webserver/scheduler 기동한다. 설치 직후 흔한 **SQLite 버전 낮음 오류**는 소스 컴파일로 최신 sqlite를 설치해 해결한다.

---

## 2. 설치

```bash
python3 -m venv py3env
source ~/py3env/bin/activate
pip install pip --upgrade
sudo yum -y install gcc gcc-c++ libffi-devel mariadb-devel
pip install apache-airflow
```

### SQLite 버전 오류 해결
`airflow version` 실행 시 sqlite 라이브러리 버전이 낮다는 오류 → 소스 빌드:
```bash
wget --no-check-certificate https://www.sqlite.org/src/tarball/sqlite.tar.gz
tar xzf sqlite.tar.gz && cd sqlite/
export CFLAGS="-DSQLITE_ENABLE_FTS3 ... -O2 -fPIC"   # 원문 옵션 참조
export PREFIX="/usr/local"
LIBS="-lm" ./configure --disable-tcl --enable-shared --enable-tempstore=always --prefix="$PREFIX"
make && sudo make install
# .bashrc 에 추가
export LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH
```

## 3. Metadb 설정 (airflow.cfg)

| DB | executor | sql_alchemy_conn |
|----|----------|------------------|
| SQLite(기본·테스트) | `SequentialExecutor` | `sqlite:////.../airflow.db` |
| **PostgreSQL(권장)** | `LocalExecutor` | `postgresql://postgres:postgres@127.0.0.1:5432` |

> scheduler 관련 오류 방지를 위해 SQLite보다 **PostgreSQL 사용 권장**.
```bash
airflow db init
```

## 4. 기동 · 계정 · 접속

```bash
airflow webserver -p 8080
airflow users create --role Admin --username admin --email admin \
  --firstname admin --lastname admin --password admin
airflow scheduler -D
# http://localhost:8080 접속
```

## 5. 환경설정 (dags 갱신주기)

```ini
# airflow.cfg
dags_folder = /home/omegaman/airflow/dags
min_file_process_interval = 1   # 신규 dag 빨리 반영
dag_dir_list_interval = 30
```

## 6. 샘플 DAG 실행

`$AIRFLOW_HOME/dags/test.py` 에 BashOperator DAG 작성 후:
```bash
airflow dags list
airflow dags test test 20220518   # CLI 직접 실행
```

## 7. 연관 개념

- [[2026-06-14-AF02_(Airflow-Docker-Compose-설치)]]
- [[2026-06-14-AF03_(Airflow-개발환경-VSCode-DevContainer)]]
